# NOVU SETUP [ NOTIFICATIONS ]
------------------------------------------------------------------------
Novu is the default notifications layer: an open-source notification
infrastructure that orchestrates multi-channel delivery (in-app, email,
SMS, push) from a single workflow, plus a prebuilt React in-app inbox.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add @novu/api @novu/react
```
`@novu/api` is the server SDK for triggering; `@novu/react` renders the
in-app inbox component.
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create an app at novu.co and copy the keys.
```text
NOVU_SECRET_KEY=xxxxxxxxxxxxxxxx
NEXT_PUBLIC_NOVU_APP_ID=xxxxxxxxxxxx
```
The secret key triggers workflows server-side; the application id is
public and identifies your app to the inbox component.
------------------------------------------------------------------------
## STEP 3 : Create a Workflow
------------------------------------------------------------------------
In the Novu dashboard, create a workflow with id `comment-added` and add
steps: an In-App step and an Email step. Reference payload variables with
`{{payload.actor}}` inside the step editor.
```text
Workflow: comment-added
  ├─ Step 1: In-App   → "{{payload.actor}} commented"
  ├─ Step 2: Delay    → 10 minutes (skip if seen)
  └─ Step 3: Email    → digest fallback if still unread
```
------------------------------------------------------------------------
## STEP 4 : Trigger from the Server
------------------------------------------------------------------------
```ts
// src/app/api/notify/route.ts
import { NextResponse } from "next/server";
import { Novu } from "@novu/api";

const novu = new Novu({ secretKey: process.env.NOVU_SECRET_KEY! });

export async function POST(req: Request) {
  const { userId, actor } = await req.json();
  await novu.trigger({
    workflowId: "comment-added",
    to: { subscriberId: userId },
    payload: { actor },
  });
  return NextResponse.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 5 : Render the In-App Inbox
------------------------------------------------------------------------
```tsx
// src/components/inbox.tsx
"use client";
import { Inbox } from "@novu/react";

export function NotificationInbox({ subscriberId }: { subscriberId: string }) {
  return (
    <Inbox
      applicationIdentifier={process.env.NEXT_PUBLIC_NOVU_APP_ID!}
      subscriberId={subscriberId}
    />
  );
}
```
The bell icon, unread count, and mark-as-read UI are all built in.
------------------------------------------------------------------------
## STEP 6 : Manage Subscribers
------------------------------------------------------------------------
```ts
// keep subscriber profile in sync (email/phone for other channels)
await novu.subscribers.create({
  subscriberId: userId,
  email: userEmail,
  firstName: "User",
});
```
Do this on signup so every channel has the contact info it needs.
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   App event ──POST /api/notify──► novu.trigger("comment-added")
                                          │
                                   Novu Workflow Engine
                    ┌──────────────┬───────────┴────────────┐
                    ▼              ▼                         ▼
                In-App         Email (SendGrid)        SMS / Push
                    │
              <Inbox/> component ── realtime unread feed ── Browser
```
Note (Windows): all packages are pure JS, so `pnpm install` needs no
build tools; self-hosting Novu locally requires Docker Desktop.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep NOVU_SECRET_KEY server-only; the inbox uses just the app id
✓ Trigger by workflowId + subscriberId; keep channel logic in the workflow
✓ Sync subscriber email/phone at signup so multi-channel works day one
✓ Use payload variables, not hardcoded copy, so editors tune templates
✓ Add delay + "skip if seen" steps to avoid notification spam
✓ Let users manage channel preferences via Novu's preference center
✓ Make triggers idempotent with a transactionId to dedupe retries
✓ Version workflows in the dashboard; test with the preview payload
```
