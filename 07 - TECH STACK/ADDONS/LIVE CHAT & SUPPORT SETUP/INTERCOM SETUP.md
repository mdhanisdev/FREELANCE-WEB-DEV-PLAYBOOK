# INTERCOM SETUP [ LIVE CHAT & SUPPORT ]
------------------------------------------------------------------------

Intercom is a full customer-engagement platform: live chat, product
tours, help center, and outbound messaging. Use it when you need
enterprise support tooling. This guide wires the Messenger into Next.js
App Router with secure Identity Verification (HMAC).

------------------------------------------------------------------------

## STEP 1 : Grab your credentials

1. Sign in at https://app.intercom.com.
2. Settings -> Installation -> Web: copy your `App ID`.
3. Settings -> Security -> Identity Verification: copy the secret key.

------------------------------------------------------------------------

## STEP 2 : Store the environment variables

```bash
# .env.local
NEXT_PUBLIC_INTERCOM_APP_ID="abcd1234"
INTERCOM_IDENTITY_SECRET="server-only-hmac-secret"
```

The App ID is public; the identity secret is server-only (never prefix
it with `NEXT_PUBLIC_`). Windows: edit `.env.local` in VS Code, not
Notepad, to avoid a BOM.

------------------------------------------------------------------------

## STEP 3 : Install the SDK

```bash
pnpm add @intercom/messenger-js-sdk
```

------------------------------------------------------------------------

## STEP 4 : Compute the HMAC on the server

```ts
// lib/intercom-hash.ts
import { createHmac } from "node:crypto";

export function intercomUserHash(userId: string): string {
  const secret = process.env.INTERCOM_IDENTITY_SECRET!;
  return createHmac("sha256", secret).update(userId).digest("hex");
}
```

Expose it through a tiny server action or route so the client never
sees the secret.

------------------------------------------------------------------------

## STEP 5 : Boot the Messenger

```tsx
// components/intercom.tsx
"use client";

import { useEffect } from "react";
import Intercom from "@intercom/messenger-js-sdk";

type Props = {
  userId: string;
  name: string;
  email: string;
  userHash: string;
};

export function IntercomWidget({ userId, name, email, userHash }: Props) {
  useEffect(() => {
    Intercom({
      app_id: process.env.NEXT_PUBLIC_INTERCOM_APP_ID!,
      user_id: userId,
      name,
      email, // e.g. user@example.com
      user_hash: userHash,
    });
  }, [userId, name, email, userHash]);

  return null;
}
```

------------------------------------------------------------------------

## STEP 6 : Trust boundary

```text
Client (public)            Server (secret)         Intercom
      |                         |                      |
      | request userHash -----> |                      |
      |                         | HMAC(userId, secret) |
      | <---- userHash -------- |                      |
      | Intercom({app_id,...user_hash}) --------------> verified session
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep INTERCOM_IDENTITY_SECRET server-only; never ship it to browsers
✓ Always compute user_hash server-side to prevent identity spoofing
✓ Only pass NEXT_PUBLIC_INTERCOM_APP_ID to the client
✓ Boot the Messenger in useEffect, guarded against SSR
✓ Call Intercom("shutdown") on logout to clear the session
✓ Gate the widget behind auth for verified, trusted conversations
✓ Reserve Intercom for when you need tours + help center at scale
```
