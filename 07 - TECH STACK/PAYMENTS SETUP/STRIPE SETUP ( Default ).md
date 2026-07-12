# STRIPE SETUP [ PAYMENTS / SUBSCRIPTIONS ]
------------------------------------------------------------------------

Stripe is a raw payment processor. You are the merchant of record, so
YOU are responsible for collecting and remitting sales tax / VAT unless
you add Stripe Tax. Use Stripe when you want maximum control over the
checkout, billing logic, and payout flow.

------------------------------------------------------------------------
## STEP 1 : Install & Env

```bash
pnpm add stripe @stripe/stripe-js
```

Create `.env.local` (never commit it):

```text
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

------------------------------------------------------------------------
## STEP 2 : Server Client

```ts
// lib/stripe.ts
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2025-06-30.basil",
  typescript: true,
});
```

------------------------------------------------------------------------
## STEP 3 : Create Checkout Session Route

```ts
// app/api/checkout/route.ts
import { NextRequest, NextResponse } from "next/server";
import { stripe } from "@/lib/stripe";

export async function POST(req: NextRequest) {
  const { priceId, userId } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: "subscription",
    line_items: [{ price: priceId, quantity: 1 }],
    // Attach identity so the webhook can grant access to the right user.
    client_reference_id: userId,
    metadata: { userId },
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
  });

  return NextResponse.json({ url: session.url });
}
```

------------------------------------------------------------------------
## STEP 4 : Redirect From The Client

```tsx
// components/CheckoutButton.tsx
"use client";

export function CheckoutButton({ priceId, userId }: { priceId: string; userId: string }) {
  async function go() {
    const res = await fetch("/api/checkout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ priceId, userId }),
    });
    const { url } = await res.json();
    window.location.href = url; // Stripe-hosted checkout page
  }
  return <button onClick={go}>Subscribe</button>;
}
```

------------------------------------------------------------------------
## STEP 5 : Webhook Handler (Raw Body + Signature Verify)

The App Router does not parse the body for you here — read the raw text
so the signature check works. Never trust the request without verifying.

```ts
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from "next/server";
import { stripe } from "@/lib/stripe";
import Stripe from "stripe";

export async function POST(req: NextRequest) {
  const body = await req.text(); // RAW body, not req.json()
  const sig = req.headers.get("stripe-signature")!;

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(body, sig, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch (err) {
    return new NextResponse(`Webhook signature failed: ${(err as Error).message}`, { status: 400 });
  }

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object as Stripe.Checkout.Session;
      const userId = session.metadata?.userId ?? session.client_reference_id;
      // GRANT ACCESS HERE — this is the source of truth, not /success.
      await grantAccess(userId, session.subscription as string);
      break;
    }
    case "customer.subscription.deleted":
      // Revoke access on cancellation / non-payment.
      break;
  }

  return NextResponse.json({ received: true });
}

async function grantAccess(userId: string | null | undefined, subId: string) {
  // Persist subscription status to your DB, keyed by userId.
}
```

------------------------------------------------------------------------
## STEP 6 : Test Webhooks Locally

```bash
# In one terminal: forward events to your local route.
stripe listen --forward-to localhost:3000/api/webhooks/stripe
# Copy the printed whsec_... into STRIPE_WEBHOOK_SECRET, then:
stripe trigger checkout.session.completed
```

------------------------------------------------------------------------
## PAYMENT FLOW

```text
  User            Next.js App        Stripe            Webhook
   |  click buy       |                 |                 |
   |----------------> | create session  |                 |
   |                  |---------------->|                 |
   |   redirect URL   |<----------------|                 |
   |<-----------------|                 |                 |
   |  pay on Stripe hosted page ------->|                 |
   |                  |    /success (UNTRUSTED, just UI)  |
   |<---- redirect ---|                 |                 |
   |                  |   checkout.session.completed ---->|
   |                  |                 |   verify sig    |
   |                  |         GRANT ACCESS in DB <------|
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Grant access from the webhook, NOT the /success page — success is just UI
✓ Always verify the signature with the RAW request body
✓ Use client_reference_id / metadata to map a session back to your user
✓ Make webhook handlers idempotent — Stripe may deliver an event twice
✓ Keep sk_test_... and whsec_... in .env.local, never in client code
✓ Add Stripe Tax if you must collect VAT/sales tax (you are merchant of record)
✓ Handle subscription.deleted / invoice.payment_failed to revoke access
✓ Test every event locally with `stripe listen` before shipping
✓ Return 2xx fast; do slow work async so Stripe does not retry
```
