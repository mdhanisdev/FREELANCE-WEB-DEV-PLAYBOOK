# ONESIGNAL SETUP [ PUSH NOTIFICATIONS ]

------------------------------------------------------------------------

OneSignal is a managed push platform. It hosts the service workers and
subscription storage; you initialise the web SDK on the client and send
notifications through its REST API from the server.

------------------------------------------------------------------------

## STEP 1 : Create App & Get Keys

In the OneSignal dashboard create a Web Push app. Copy the **App ID** and
a **REST API Key**.

```bash
pnpm add react-onesignal
```

```text
.env.local
NEXT_PUBLIC_ONESIGNAL_APP_ID=00000000-0000-0000-0000-000000000000
ONESIGNAL_REST_API_KEY=your_rest_api_key_placeholder
```

------------------------------------------------------------------------

## STEP 2 : Host The SDK Workers

Download the OneSignal SDK worker files and place them in `public/` so
they are served from the root scope.

```text
public/
  OneSignalSDKWorker.js        (imports OneSignal SDK)
  OneSignalSDKUpdaterWorker.js
```

------------------------------------------------------------------------

## STEP 3 : Initialise On The Client

```tsx
"use client";
import { useEffect } from "react";
import OneSignal from "react-onesignal";

export function PushInit() {
  useEffect(() => {
    OneSignal.init({
      appId: process.env.NEXT_PUBLIC_ONESIGNAL_APP_ID!,
      allowLocalhostAsSecureOrigin: true,
    }).then(() => OneSignal.Slidedown.promptPush());
  }, []);
  return null;
}
```

------------------------------------------------------------------------

## STEP 4 : Send From The Server

```ts
// app/api/push/send/route.ts
import { NextResponse } from "next/server";

export async function POST() {
  const res = await fetch("https://api.onesignal.com/notifications", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Key ${process.env.ONESIGNAL_REST_API_KEY}`,
    },
    body: JSON.stringify({
      app_id: process.env.NEXT_PUBLIC_ONESIGNAL_APP_ID,
      included_segments: ["Subscribed Users"],
      headings: { en: "Hello" },
      contents: { en: "New activity in your inbox" },
      url: "https://example.com/inbox",
    }),
  });

  if (!res.ok) {
    return NextResponse.json({ error: "send failed" }, { status: 502 });
  }
  return NextResponse.json({ sent: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Architecture

```text
  Client                OneSignal                Your Server
  ------                ---------                -----------
  OneSignal.init  --->  stores subscription
  prompt + opt-in
                                    <----- POST /notifications ----
                        fan-out to segment          (REST API Key)
  browser notification <----------
```

OneSignal owns subscription storage; you target users by segment,
external user ID, or player ID rather than raw push subscriptions.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`allowLocalhostAsSecureOrigin: true` lets you test over `http://localhost`
on Windows during `pnpm dev`; production still requires HTTPS.
Notifications render via the Windows Action Center and are suppressed by
focus assist. Keep worker filenames exact — Windows is case-insensitive
but deployment targets often are not.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep the REST API Key server-only; only the App ID is public
✓ Prompt for permission after a user action, not on first paint
✓ Use external user IDs so pushes follow users across devices
✓ Segment sends; avoid blasting "Subscribed Users" for everything
✓ Handle non-2xx REST responses and retry transient failures
✓ Keep both SDK worker files in public/ at the site root
✓ Provide an easy in-app opt-out to reduce spam complaints
```
