# KNOCK SETUP [ NOTIFICATIONS ]
------------------------------------------------------------------------
Knock is a hosted notifications platform that turns product events into
cross-channel messages (in-app feed, email, SMS, push, Slack) via
visual workflows. It ships a prebuilt React in-app feed and a preference
center, driven by short-lived user tokens.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add @knocklabs/node
pnpm add @knocklabs/react
```
`@knocklabs/node` triggers workflows server-side; `@knocklabs/react`
renders the feed and notification UI.
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create an account at knock.app and copy the keys.
```text
KNOCK_API_KEY=sk_xxxxxxxxxxxx
NEXT_PUBLIC_KNOCK_PUBLIC_API_KEY=pk_xxxxxxxxxxxx
NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID=xxxxxxxx-xxxx-xxxx
```
The secret key is server-only; the public key + feed channel id are safe
in the browser.
------------------------------------------------------------------------
## STEP 3 : Trigger a Workflow
------------------------------------------------------------------------
```ts
// src/app/api/notify/route.ts
import { NextResponse } from "next/server";
import { Knock } from "@knocklabs/node";

const knock = new Knock({ apiKey: process.env.KNOCK_API_KEY! });

export async function POST(req: Request) {
  const { userId, actor } = await req.json();
  await knock.workflows.trigger("comment-added", {
    recipients: [userId],
    data: { actor },
  });
  return NextResponse.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 4 : Identify Users
------------------------------------------------------------------------
```ts
// sync recipient profile so email/SMS channels have contact info
await knock.users.identify(userId, {
  email: userEmail,
  name: "User",
});
```
Call this on signup and whenever the profile changes.
------------------------------------------------------------------------
## STEP 5 : Render the In-App Feed
------------------------------------------------------------------------
```tsx
// src/components/feed.tsx
"use client";
import {
  KnockProvider,
  KnockFeedProvider,
  NotificationIconButton,
  NotificationFeedPopover,
} from "@knocklabs/react";
import { useRef, useState } from "react";
import "@knocklabs/react/dist/index.css";

export function Feed({ userId, token }: { userId: string; token: string }) {
  const [open, setOpen] = useState(false);
  const ref = useRef(null);
  return (
    <KnockProvider
      apiKey={process.env.NEXT_PUBLIC_KNOCK_PUBLIC_API_KEY!}
      userId={userId}
      userToken={token}
    >
      <KnockFeedProvider feedId={process.env.NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID!}>
        <NotificationIconButton ref={ref} onClick={() => setOpen((o) => !o)} />
        <NotificationFeedPopover
          buttonRef={ref}
          isVisible={open}
          onClose={() => setOpen(false)}
        />
      </KnockFeedProvider>
    </KnockProvider>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Sign the User Token (Server)
------------------------------------------------------------------------
```ts
// enhanced security mode: sign a short-lived JWT per user
import { Knock } from "@knocklabs/node";
const token = await Knock.signUserToken(userId, {
  signingKey: process.env.KNOCK_SIGNING_KEY!,
  expiresInSeconds: 3600,
});
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   App event ──POST /api/notify──► knock.workflows.trigger()
                                          │
                                   Knock Workflow Engine
                    ┌──────────────┬──────────┴─────────────┐
                    ▼              ▼                         ▼
                In-App Feed    Email                   SMS / Slack / Push
                    │
        <KnockFeedProvider/> ── signed user token ── Browser
```
Note (Windows): packages are pure JS with no native build step; ensure
the signing key file uses LF newlines if loaded from disk.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep the secret + signing keys server-only; browser gets the public key
✓ Enable enhanced security and sign short-lived user tokens
✓ Identify users (email/phone) before triggering multi-channel workflows
✓ Pass structured data to triggers; keep copy in Knock templates
✓ Use per-user preferences so recipients control channels and frequency
✓ Batch/throttle noisy events inside the workflow, not in app code
✓ Deduplicate with a cancellation key for retried triggers
✓ Import the feed CSS once and theme via the provider, not overrides
```
