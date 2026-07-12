# SVIX SETUP [ WEBHOOKS ]
------------------------------------------------------------------------
Svix is our default webhook infrastructure: it handles sending (retries,
signing, fan-out) so you send an event and it delivers to your customers'
endpoints. This guide covers both sending from Next.js and verifying
inbound webhooks, using TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add svix
```
------------------------------------------------------------------------
## STEP 2 : Configure Environment

```bash
# .env.local
SVIX_API_KEY=sk_your_svix_api_key
SVIX_WEBHOOK_SECRET=whsec_your_endpoint_secret
```

```text
Sending vs receiving
  YOUR APP ──event──▶ SVIX ──signed POST──▶ CUSTOMER endpoint
  PROVIDER ──signed POST──▶ YOUR /api/webhooks ──verify──▶ handle
```
------------------------------------------------------------------------
## STEP 3 : Send an Event (Outbound)

```ts
// lib/svix.ts
import { Svix } from "svix";

export const svix = new Svix(process.env.SVIX_API_KEY!);

export async function sendEvent(appId: string, type: string, data: unknown) {
  return svix.message.create(appId, {
    eventType: type,
    payload: { type, data },
  });
}
```

```ts
// app/api/orders/route.ts
import { sendEvent } from "@/lib/svix";

export async function POST() {
  // ...create order...
  await sendEvent("app_123", "order.created", { id: "ord_1" });
  return Response.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 4 : Receive & Verify (Inbound)

Read the raw body — parsing first breaks signature verification.

```ts
// app/api/webhooks/route.ts
import { Webhook } from "svix";

export async function POST(req: Request) {
  const payload = await req.text(); // RAW body, not req.json()
  const headers = {
    "svix-id": req.headers.get("svix-id")!,
    "svix-timestamp": req.headers.get("svix-timestamp")!,
    "svix-signature": req.headers.get("svix-signature")!,
  };

  const wh = new Webhook(process.env.SVIX_WEBHOOK_SECRET!);
  let evt: { type: string; data: unknown };
  try {
    evt = wh.verify(payload, headers) as typeof evt;
  } catch {
    return new Response("invalid signature", { status: 400 });
  }

  switch (evt.type) {
    case "order.created":
      // handle...
      break;
  }
  return Response.json({ received: true });
}
```
------------------------------------------------------------------------
## STEP 5 : Test the Endpoint

```bash
pnpm add -D svix-cli
pnpm dlx svix listen http://localhost:3000/api/webhooks
```

The listen command gives you a public URL that forwards signed test
events to your local route.
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# trigger an event, confirm 200 + "received" in logs
```

Windows note: `req.text()` returns identical bytes on Windows; do not
normalize CRLF/LF on the raw payload or the signature check will fail.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Verify every inbound webhook signature before trusting it
✓ Read the RAW request body; never JSON.parse before verifying
✓ Reject on signature failure with a 4xx, never process it
✓ Return 2xx fast; offload slow work to a queue
✓ Make handlers idempotent — deliveries can repeat
✓ Rotate endpoint secrets and store them server-side only
✓ Let Svix own retries/backoff instead of hand-rolling them
✓ Log svix-id to dedupe and trace individual deliveries
```
