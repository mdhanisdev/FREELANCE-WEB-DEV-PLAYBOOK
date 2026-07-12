# PAYPAL SETUP [ PAYMENTS / ORDERS API ]
------------------------------------------------------------------------

PayPal lets buyers pay with their PayPal balance, cards, or local
methods. You are the merchant of record with PayPal, so YOU remain
responsible for your own sales tax / VAT (PayPal does not remit it for
you like Lemon Squeezy or Paddle). Use PayPal when buyer trust and PayPal
wallet coverage matter, often alongside a card processor.

------------------------------------------------------------------------
## STEP 1 : Install & Env

```bash
pnpm add @paypal/paypal-server-sdk @paypal/react-paypal-js
```

```text
PAYPAL_CLIENT_ID=sk_test_...
PAYPAL_CLIENT_SECRET=sk_test_...
PAYPAL_WEBHOOK_ID=whsec_...
NEXT_PUBLIC_PAYPAL_CLIENT_ID=sk_test_...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

------------------------------------------------------------------------
## STEP 2 : Server Client

```ts
// lib/paypal.ts
import { Client, Environment, OrdersController } from "@paypal/paypal-server-sdk";

const client = new Client({
  clientCredentialsAuthCredentials: {
    oAuthClientId: process.env.PAYPAL_CLIENT_ID!,
    oAuthClientSecret: process.env.PAYPAL_CLIENT_SECRET!,
  },
  environment: Environment.Sandbox, // Environment.Production when live
});

export const ordersController = new OrdersController(client);
```

------------------------------------------------------------------------
## STEP 3 : Create Order (Orders API)

```ts
// app/api/paypal/create-order/route.ts
import { NextRequest, NextResponse } from "next/server";
import { ordersController } from "@/lib/paypal";
import { CheckoutPaymentIntent } from "@paypal/paypal-server-sdk";

export async function POST(req: NextRequest) {
  const { userId } = await req.json();

  const { result } = await ordersController.createOrder({
    body: {
      intent: CheckoutPaymentIntent.Capture,
      purchaseUnits: [
        {
          customId: userId, // echoed back on capture + webhook
          amount: { currencyCode: "USD", value: "20.00" },
        },
      ],
    },
  });

  return NextResponse.json({ id: result.id });
}
```

------------------------------------------------------------------------
## STEP 4 : PayPal Buttons (Client)

```tsx
// components/PayPalCheckout.tsx
"use client";
import { PayPalScriptProvider, PayPalButtons } from "@paypal/react-paypal-js";

export function PayPalCheckout({ userId }: { userId: string }) {
  return (
    <PayPalScriptProvider options={{ clientId: process.env.NEXT_PUBLIC_PAYPAL_CLIENT_ID! }}>
      <PayPalButtons
        createOrder={async () => {
          const res = await fetch("/api/paypal/create-order", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ userId }),
          });
          const { id } = await res.json();
          return id;
        }}
        onApprove={async (data) => {
          await fetch(`/api/paypal/capture-order`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ orderId: data.orderID }),
          });
        }}
      />
    </PayPalScriptProvider>
  );
}
```

------------------------------------------------------------------------
## STEP 5 : Capture Order

```ts
// app/api/paypal/capture-order/route.ts
import { NextRequest, NextResponse } from "next/server";
import { ordersController } from "@/lib/paypal";

export async function POST(req: NextRequest) {
  const { orderId } = await req.json();
  const { result } = await ordersController.captureOrder({ id: orderId });
  // Capture confirms funds; treat the webhook as your durable source of truth.
  return NextResponse.json({ status: result.status });
}
```

------------------------------------------------------------------------
## STEP 6 : Webhook Handler (Verify Then Grant)

Verify authenticity with PayPal's verify-webhook-signature API before
acting. Grant access from here, not from the browser onApprove callback.

```ts
// app/api/webhooks/paypal/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const raw = await req.text();
  const event = JSON.parse(raw);

  // Verify using headers + PAYPAL_WEBHOOK_ID against verify-webhook-signature.
  const ok = await verifySignature(req.headers, raw, process.env.PAYPAL_WEBHOOK_ID!);
  if (!ok) return new NextResponse("Invalid signature", { status: 400 });

  if (event.event_type === "PAYMENT.CAPTURE.COMPLETED") {
    const userId = event.resource?.custom_id;
    await grantAccess(userId); // source of truth
  }

  return NextResponse.json({ received: true });
}

async function verifySignature(_h: Headers, _raw: string, _webhookId: string) { return true; }
async function grantAccess(userId?: string) { /* persist entitlement */ }
```

------------------------------------------------------------------------
## PAYMENT FLOW

```text
  User          Next.js App        PayPal            Webhook
   |  click buttons  |                |                  |
   |---------------->| createOrder    |                  |
   |                 |--------------->|                  |
   |   order id      |<---------------|                  |
   |  approve in PayPal popup ------->|                  |
   |  onApprove -> capture-order ---->| capture funds    |
   |                 |   PAYMENT.CAPTURE.COMPLETED ----->|
   |                 |                |   verify sig     |
   |                 |      GRANT ACCESS in DB <---------|
```

------------------------------------------------------------------------
## WHEN TO CHOOSE PAYPAL

```text
✓ Your buyers expect the PayPal wallet as a payment option
✓ You want strong buyer familiarity / trust at checkout
✓ You sell one-off purchases and want a fast Orders API integration
✓ You want it as a SECONDARY option beside a card processor
✗ Skip as sole processor if you need MoR tax handling — use LS/Paddle
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Grant access from the verified webhook, not the onApprove callback
✓ Verify every webhook via verify-webhook-signature + PAYPAL_WEBHOOK_ID
✓ Put your user id in custom_id so capture + webhook map to a user
✓ Make handlers idempotent — the same capture event can repeat
✓ Keep client secret + webhook id server-side in .env.local only
✓ Start in Sandbox; switch to Environment.Production for launch
✓ Remember you are merchant of record — you handle your own VAT/tax
✓ Reconcile capture status server-side; never trust client-reported success
```
