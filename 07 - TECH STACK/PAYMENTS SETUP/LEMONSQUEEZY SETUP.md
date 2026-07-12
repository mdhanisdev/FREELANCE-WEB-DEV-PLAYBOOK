# LEMON SQUEEZY SETUP [ PAYMENTS / MERCHANT OF RECORD ]
------------------------------------------------------------------------

Lemon Squeezy is a Merchant of Record (MoR). It sells to your customer
on your behalf, which means Lemon Squeezy calculates, collects, and
remits sales tax / VAT / GST for you worldwide. Unlike raw Stripe, you
do NOT have to register for tax in each jurisdiction — that liability is
theirs. Great for solo devs and small teams selling globally.

------------------------------------------------------------------------
## STEP 1 : Install & Env

```bash
pnpm add @lemonsqueezy/lemonsqueezy.js
```

```text
LEMONSQUEEZY_API_KEY=sk_test_...
LEMONSQUEEZY_STORE_ID=12345
LEMONSQUEEZY_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

------------------------------------------------------------------------
## STEP 2 : Server Client

```ts
// lib/lemonsqueezy.ts
import { lemonSqueezySetup } from "@lemonsqueezy/lemonsqueezy.js";

lemonSqueezySetup({
  apiKey: process.env.LEMONSQUEEZY_API_KEY!,
  onError: (err) => console.error("Lemon Squeezy error", err),
});
```

------------------------------------------------------------------------
## STEP 3 : Create A Checkout

```ts
// app/api/checkout/route.ts
import { NextRequest, NextResponse } from "next/server";
import { createCheckout } from "@lemonsqueezy/lemonsqueezy.js";
import "@/lib/lemonsqueezy";

export async function POST(req: NextRequest) {
  const { variantId, userId, email } = await req.json();

  const checkout = await createCheckout(
    process.env.LEMONSQUEEZY_STORE_ID!,
    variantId,
    {
      checkoutData: {
        email,
        // custom data comes back on every webhook — map to your user.
        custom: { user_id: userId },
      },
      productOptions: {
        redirectUrl: `${process.env.NEXT_PUBLIC_APP_URL}/success`,
      },
    },
  );

  return NextResponse.json({ url: checkout.data?.data.attributes.url });
}
```

------------------------------------------------------------------------
## STEP 4 : Redirect From The Client

```tsx
// components/BuyButton.tsx
"use client";

export function BuyButton({ variantId, userId, email }: { variantId: string; userId: string; email: string }) {
  async function go() {
    const res = await fetch("/api/checkout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ variantId, userId, email }),
    });
    const { url } = await res.json();
    window.location.href = url;
  }
  return <button onClick={go}>Buy now</button>;
}
```

------------------------------------------------------------------------
## STEP 5 : Webhook Handler (HMAC SHA-256, Raw Body)

Lemon Squeezy signs the raw body with your webhook secret. Verify it.

```ts
// app/api/webhooks/lemonsqueezy/route.ts
import { NextRequest, NextResponse } from "next/server";
import crypto from "node:crypto";

export async function POST(req: NextRequest) {
  const raw = await req.text(); // RAW body for HMAC
  const signature = req.headers.get("x-signature") ?? "";

  const hmac = crypto.createHmac("sha256", process.env.LEMONSQUEEZY_WEBHOOK_SECRET!);
  const digest = hmac.update(raw).digest("hex");

  if (!crypto.timingSafeEqual(Buffer.from(digest), Buffer.from(signature))) {
    return new NextResponse("Invalid signature", { status: 400 });
  }

  const event = JSON.parse(raw);
  const name = event.meta.event_name;
  const userId = event.meta.custom_data?.user_id;

  switch (name) {
    case "order_created":
    case "subscription_created":
      await grantAccess(userId); // source of truth — grant here
      break;
    case "subscription_cancelled":
    case "subscription_expired":
      // revoke access
      break;
  }

  return NextResponse.json({ received: true });
}

async function grantAccess(userId?: string) {
  // Persist entitlement keyed by userId.
}
```

------------------------------------------------------------------------
## PAYMENT FLOW

```text
  User          Next.js App      Lemon Squeezy (MoR)     Webhook
   |  buy            |                  |                   |
   |---------------->| createCheckout   |                   |
   |                 |----------------->|                   |
   |  hosted URL     |<-----------------|                   |
   |<----------------|                  |                   |
   |  pay (LS collects VAT/tax) ------->|                   |
   |                 |   order_created / subscription ----->|
   |                 |                  |   verify HMAC     |
   |                 |        GRANT ACCESS in DB <----------|
```

------------------------------------------------------------------------
## WHEN TO CHOOSE LEMON SQUEEZY

```text
✓ You sell globally and do NOT want to handle VAT/sales tax yourself
✓ You are a solo dev / small team wanting the least tax + compliance work
✓ You sell digital products, SaaS subscriptions, or license keys
✓ You are fine with LS taking a slightly higher fee in exchange for MoR
✗ Skip it if you need deep custom billing logic — reach for raw Stripe
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Remember: Lemon Squeezy handles sales tax/VAT for you (unlike raw Stripe)
✓ Grant access from the webhook, not the redirect/success page
✓ Verify the x-signature HMAC against the RAW body with timingSafeEqual
✓ Pass your user_id in checkoutData.custom so webhooks map to a user
✓ Make handlers idempotent — the same event can arrive more than once
✓ Keep sk_test_... and whsec_... in .env.local only
✓ Handle subscription_cancelled / _expired to revoke access
✓ Return 2xx quickly; defer heavy work so LS does not retry
```
