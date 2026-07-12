# CAL.COM EMBED SETUP [ CALENDAR & SCHEDULING ]

------------------------------------------------------------------------

Cal.com is a hosted (or self-hosted) scheduling platform. Instead of
building booking logic, you embed a Cal.com booking page and react to
booking events via its embed API and webhooks.

------------------------------------------------------------------------

## STEP 1 : Create A Booking Link

Sign up at Cal.com, create an event type, and note your public link, e.g.
`example-user/intro-call`. Grab an API key for webhook verification.

```bash
pnpm add @calcom/embed-react
```

```text
.env.local
NEXT_PUBLIC_CAL_LINK=example-user/intro-call
CAL_WEBHOOK_SECRET=your_webhook_secret_placeholder
```

------------------------------------------------------------------------

## STEP 2 : Embed The Booker (Client)

```tsx
"use client";
import Cal, { getCalApi } from "@calcom/embed-react";
import { useEffect } from "react";

export function BookingWidget() {
  useEffect(() => {
    (async () => {
      const cal = await getCalApi();
      cal("ui", { theme: "light", hideEventTypeDetails: false });
    })();
  }, []);

  return (
    <Cal
      calLink={process.env.NEXT_PUBLIC_CAL_LINK!}
      style={{ width: "100%", height: "100%", minHeight: 600 }}
    />
  );
}
```

------------------------------------------------------------------------

## STEP 3 : React To Booking Events

The embed emits browser events you can subscribe to for analytics or UI.

```tsx
"use client";
import { getCalApi } from "@calcom/embed-react";
import { useEffect } from "react";

export function BookingListener() {
  useEffect(() => {
    (async () => {
      const cal = await getCalApi();
      cal("on", {
        action: "bookingSuccessful",
        callback: (e) => console.log("booked", e.detail),
      });
    })();
  }, []);
  return null;
}
```

------------------------------------------------------------------------

## STEP 4 : Handle The Webhook (Server)

Cal.com posts a signed payload when a booking is created. Verify it.

```ts
// app/api/cal/webhook/route.ts
import { NextRequest, NextResponse } from "next/server";
import { createHmac, timingSafeEqual } from "crypto";

export async function POST(req: NextRequest) {
  const raw = await req.text();
  const sig = req.headers.get("x-cal-signature-256") ?? "";
  const expected = createHmac("sha256", process.env.CAL_WEBHOOK_SECRET!)
    .update(raw)
    .digest("hex");

  if (!timingSafeEqual(Buffer.from(sig), Buffer.from(expected))) {
    return NextResponse.json({ error: "bad signature" }, { status: 401 });
  }

  const event = JSON.parse(raw);
  if (event.triggerEvent === "BOOKING_CREATED") {
    await db.bookings.create({ data: event.payload });
  }
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Integration Flow

```text
  Browser                Cal.com               Your Server
  -------                -------               -----------
  <Cal calLink=.. />     hosts availability
  user books ------->    creates booking
       'bookingSuccessful' event (client UI)
                         BOOKING_CREATED ----->  /api/cal/webhook
                                                 verify sig -> persist
```

The client event is for UX; the webhook is the source of truth.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` behaves identically on Windows. Webhooks cannot reach
`localhost` directly — use a tunnel such as `pnpm dlx localtunnel --port
3000` (or the ngrok Windows build) so Cal.com can deliver events while
you develop.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Treat the webhook as the source of truth, not the client event
✓ Verify the x-cal-signature-256 HMAC with timingSafeEqual
✓ Read the raw body before parsing so the signature stays valid
✓ Keep the webhook secret server-only
✓ Make webhook handling idempotent (dedupe by booking uid)
✓ Set the embed theme to match your app for a seamless look
✓ Tunnel localhost during development so bookings reach you
```
