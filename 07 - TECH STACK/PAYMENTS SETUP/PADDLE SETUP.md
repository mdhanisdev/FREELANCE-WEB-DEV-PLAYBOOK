# PADDLE SETUP [ PAYMENTS / MERCHANT OF RECORD ]
------------------------------------------------------------------------

Paddle is a Merchant of Record (MoR) built around Paddle Billing. Like
Lemon Squeezy, Paddle sells to your customer on your behalf and handles
sales tax / VAT / GST collection and remittance for you globally. Unlike
raw Stripe, you are NOT the merchant of record and carry no tax filing
burden per jurisdiction. Paddle scales well for established SaaS.

------------------------------------------------------------------------
## STEP 1 : Install & Env

```bash
pnpm add @paddle/paddle-node-sdk @paddle/paddle-js
```

```text
PADDLE_API_KEY=sk_test_...
PADDLE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_PADDLE_CLIENT_TOKEN=test_...
NEXT_PUBLIC_PADDLE_ENV=sandbox
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

------------------------------------------------------------------------
## STEP 2 : Server Client

```ts
// lib/paddle.ts
import { Paddle, Environment } from "@paddle/paddle-node-sdk";

export const paddle = new Paddle(process.env.PADDLE_API_KEY!, {
  environment: Environment.sandbox, // switch to .production when live
});
```

------------------------------------------------------------------------
## STEP 3 : Open Checkout (Client Overlay)

Paddle Billing renders an inline/overlay checkout via Paddle.js.

```tsx
// components/PaddleCheckout.tsx
"use client";
import { useEffect } from "react";
import { initializePaddle, Paddle } from "@paddle/paddle-js";

let paddle: Paddle | undefined;

export function PaddleCheckout({ priceId, userId }: { priceId: string; userId: string }) {
  useEffect(() => {
    initializePaddle({
      environment: process.env.NEXT_PUBLIC_PADDLE_ENV as "sandbox" | "production",
      token: process.env.NEXT_PUBLIC_PADDLE_CLIENT_TOKEN!,
    }).then((p) => (paddle = p));
  }, []);

  function open() {
    paddle?.Checkout.open({
      items: [{ priceId, quantity: 1 }],
      // customData travels through to your webhooks — map to your user.
      customData: { user_id: userId },
      settings: { successUrl: `${process.env.NEXT_PUBLIC_APP_URL}/success` },
    });
  }

  return <button onClick={open}>Subscribe</button>;
}
```

------------------------------------------------------------------------
## STEP 4 : Webhook Handler (Signature Verify, Raw Body)

The Node SDK verifies the `paddle-signature` header against the raw body.

```ts
// app/api/webhooks/paddle/route.ts
import { NextRequest, NextResponse } from "next/server";
import { paddle } from "@/lib/paddle";

export async function POST(req: NextRequest) {
  const raw = await req.text(); // RAW body for verification
  const signature = req.headers.get("paddle-signature") ?? "";

  let event;
  try {
    event = await paddle.webhooks.unmarshal(raw, process.env.PADDLE_WEBHOOK_SECRET!, signature);
  } catch {
    return new NextResponse("Invalid signature", { status: 400 });
  }

  const userId = (event?.data as any)?.customData?.user_id;

  switch (event?.eventType) {
    case "transaction.completed":
    case "subscription.activated":
      await grantAccess(userId); // source of truth — grant here
      break;
    case "subscription.canceled":
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
  User          Next.js App        Paddle (MoR)         Webhook
   |  click        |                    |                   |
   |-------------->| Paddle.Checkout.open()                 |
   |  overlay checkout (Paddle collects VAT/tax) ---------->|
   |  pay          |                    |                   |
   |               |   transaction.completed / activated -->|
   |               |                    |   verify sig      |
   |               |         GRANT ACCESS in DB <-----------|
   |  successUrl (UI only, do NOT grant here)               |
```

------------------------------------------------------------------------
## WHEN TO CHOOSE PADDLE

```text
✓ You run a SaaS and want tax/VAT compliance fully handled (MoR)
✓ You want a mature billing platform with dunning, proration, retries
✓ You sell globally and want one platform to own tax remittance
✓ You want an overlay checkout that stays on your own domain
✗ Skip it if you need bespoke payout/marketplace logic — use raw Stripe
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Remember: Paddle handles sales tax/VAT for you (unlike raw Stripe)
✓ Grant access from the webhook, not the successUrl page
✓ Verify paddle-signature with paddle.webhooks.unmarshal on the RAW body
✓ Pass customData.user_id so webhook events map back to your user
✓ Make handlers idempotent — events may be delivered more than once
✓ Keep sk_test_... and whsec_... server-side in .env.local only
✓ Start in sandbox; flip to Environment.production for launch
✓ Handle subscription.canceled / past_due to revoke access
```
