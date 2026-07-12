# FIREBASE FCM SETUP [ PUSH NOTIFICATIONS ]

------------------------------------------------------------------------

Firebase Cloud Messaging delivers web push through a Firebase service
worker. The client fetches an FCM token, you store it, and the server
sends messages with the Firebase Admin SDK.

------------------------------------------------------------------------

## STEP 1 : Project & Credentials

In the Firebase console create a project, enable Cloud Messaging, and
generate a Web Push (VAPID) key pair plus an Admin service-account JSON.

```bash
pnpm add firebase firebase-admin
```

```text
.env.local
NEXT_PUBLIC_FIREBASE_API_KEY=AIzaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_FIREBASE_PROJECT_ID=example-app
NEXT_PUBLIC_FIREBASE_VAPID_KEY=BPxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
FIREBASE_SERVICE_ACCOUNT=./service-account.json
```

------------------------------------------------------------------------

## STEP 2 : Firebase Messaging Service Worker

FCM requires this exact filename at the site root.

```ts
// public/firebase-messaging-sw.js
importScripts("https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js");
importScripts("https://www.gstatic.com/firebasejs/10.12.0/firebase-messaging-compat.js");

firebase.initializeApp({
  apiKey: "AIzaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  projectId: "example-app",
  messagingSenderId: "000000000000",
  appId: "1:000000000000:web:xxxxxxxxxxxx",
});

const messaging = firebase.messaging();
messaging.onBackgroundMessage((payload) => {
  self.registration.showNotification(payload.notification.title, {
    body: payload.notification.body,
  });
});
```

------------------------------------------------------------------------

## STEP 3 : Get A Token (Client)

```tsx
"use client";
import { getMessaging, getToken } from "firebase/messaging";
import { initializeApp } from "firebase/app";

export async function registerFcm() {
  const app = initializeApp({
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY!,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID!,
  });
  const token = await getToken(getMessaging(app), {
    vapidKey: process.env.NEXT_PUBLIC_FIREBASE_VAPID_KEY!,
  });
  await fetch("/api/push/register", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ token }),
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Send From The Server

```ts
// app/api/push/send/route.ts
import { NextResponse } from "next/server";
import { cert, getApps, initializeApp } from "firebase-admin/app";
import { getMessaging } from "firebase-admin/messaging";

if (!getApps().length) {
  initializeApp({ credential: cert(process.env.FIREBASE_SERVICE_ACCOUNT!) });
}

export async function POST() {
  const token = await getStoredToken(); // your DB lookup
  await getMessaging().send({
    token,
    notification: { title: "Hello", body: "New activity in your inbox" },
  });
  return NextResponse.json({ sent: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Token Flow

```text
  Client                 FCM                  Your Server
  ------                 ---                  -----------
  getToken(vapidKey) -> registration token
  POST /register --------------------------->  store token
                                    send(token) <--- Admin SDK
  browser <---- push ---- FCM ---------------
  sw onBackgroundMessage -> showNotification
```

Delete tokens that return `messaging/registration-token-not-registered`.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`localhost` counts as a secure origin, so `pnpm dev` works on Windows
without a certificate. Keep `firebase-messaging-sw.js` at the web root
with that exact name. Store the service-account JSON outside the repo and
reference it via an env path; never commit it.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep the service-account JSON server-only and out of git
✓ Use the exact firebase-messaging-sw.js filename at root scope
✓ Persist tokens per device and prune unregistered ones
✓ Handle onMessage (foreground) and onBackgroundMessage separately
✓ Refresh tokens periodically; they can rotate
✓ Prompt for permission on a user gesture, not on load
✓ Prefer data-only messages when you need custom rendering logic
```
