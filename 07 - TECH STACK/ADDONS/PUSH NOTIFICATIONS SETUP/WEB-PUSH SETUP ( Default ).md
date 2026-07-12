# WEB PUSH SETUP [ PUSH NOTIFICATIONS ]

------------------------------------------------------------------------

Web Push uses the browser Push API + a service worker + VAPID keys to
deliver notifications with no third-party SDK. The client subscribes, you
store the subscription, and your server signs pushes with the VAPID keys.

------------------------------------------------------------------------

## STEP 1 : Generate VAPID Keys

```bash
pnpm add web-push
pnpm dlx web-push generate-vapid-keys
```

```text
.env.local
NEXT_PUBLIC_VAPID_PUBLIC_KEY=BPxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxPUBLIC
VAPID_PRIVATE_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxPRIVATE
VAPID_SUBJECT=mailto:user@example.com
```

------------------------------------------------------------------------

## STEP 2 : Add A Service Worker

Place it in `public/` so it is served from the site root scope.

```ts
// public/sw.js
self.addEventListener("push", (event) => {
  const data = event.data ? event.data.json() : {};
  event.waitUntil(
    self.registration.showNotification(data.title ?? "Update", {
      body: data.body ?? "",
      icon: "/icon-192.png",
      data: { url: data.url ?? "/" },
    }),
  );
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow(event.notification.data.url));
});
```

------------------------------------------------------------------------

## STEP 3 : Subscribe The Client

```tsx
"use client";
export async function subscribe() {
  const reg = await navigator.serviceWorker.register("/sw.js");
  const sub = await reg.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!,
  });
  await fetch("/api/push/subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(sub),
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Send A Push (Server)

Store subscriptions in your DB; this handler just sends to one.

```ts
// app/api/push/send/route.ts
import { NextResponse } from "next/server";
import webpush from "web-push";

webpush.setVapidDetails(
  process.env.VAPID_SUBJECT!,
  process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!,
);

export async function POST() {
  const subscription = await getStoredSubscription(); // your DB lookup
  await webpush.sendNotification(
    subscription,
    JSON.stringify({ title: "Hello", body: "New activity", url: "/inbox" }),
  );
  return NextResponse.json({ sent: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Delivery Flow

```text
  Browser              Your Server           Push Service
  -------              -----------           ------------
  register sw
  subscribe --------->  store subscription
                        sendNotification --->  (VAPID-signed)
                                               push to browser
  sw 'push' event <----------------------------------
  showNotification()
```

Handle `410 Gone` from the push service by deleting the dead subscription.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

Push requires HTTPS except on `localhost`, which browsers treat as secure
even on Windows — so `pnpm dev` works locally. Notifications surface
through the Windows Action Center; focus-assist / do-not-disturb can
suppress them silently during testing.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Request notification permission from a user gesture, not on load
✓ Store one subscription per device and prune 404/410 responses
✓ Keep VAPID_PRIVATE_KEY server-only; public key can ship to client
✓ Keep payloads small and always set a title + body fallback
✓ Version the service worker file to force clean updates
✓ Always call event.waitUntil in sw handlers to avoid early exit
✓ Test with focus-assist off so notifications actually render
```
