# PUSHER SETUP [ REALTIME ]
------------------------------------------------------------------------
Pusher Channels is the default realtime layer: a hosted pub/sub service
with a simple channel model, presence, and private channels. The server
triggers events; browsers subscribe. No socket infra to run yourself.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add pusher pusher-js
```
`pusher` is the Node server SDK; `pusher-js` is the browser client.
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create an app at dashboard.pusher.com and copy the credentials.
```text
PUSHER_APP_ID=xxxxxx
PUSHER_SECRET=xxxxxxxxxxxx
NEXT_PUBLIC_PUSHER_KEY=xxxxxxxxxxxx
NEXT_PUBLIC_PUSHER_CLUSTER=eu
```
Only the key and cluster are public; app id and secret stay server-side.
------------------------------------------------------------------------
## STEP 3 : Server Client & Trigger
------------------------------------------------------------------------
```ts
// src/lib/pusher-server.ts
import Pusher from "pusher";

export const pusher = new Pusher({
  appId: process.env.PUSHER_APP_ID!,
  key: process.env.NEXT_PUBLIC_PUSHER_KEY!,
  secret: process.env.PUSHER_SECRET!,
  cluster: process.env.NEXT_PUBLIC_PUSHER_CLUSTER!,
  useTLS: true,
});
```
```ts
// src/app/api/message/route.ts
import { NextResponse } from "next/server";
import { pusher } from "@/lib/pusher-server";

export async function POST(req: Request) {
  const { text } = await req.json();
  await pusher.trigger("chat", "new-message", { text, at: Date.now() });
  return NextResponse.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 4 : Browser Client Hook
------------------------------------------------------------------------
```tsx
// src/hooks/use-channel.ts
"use client";
import { useEffect, useRef } from "react";
import Pusher from "pusher-js";

export function useChannel(channel: string, event: string, cb: (data: unknown) => void) {
  const saved = useRef(cb);
  saved.current = cb;
  useEffect(() => {
    const client = new Pusher(process.env.NEXT_PUBLIC_PUSHER_KEY!, {
      cluster: process.env.NEXT_PUBLIC_PUSHER_CLUSTER!,
    });
    const ch = client.subscribe(channel);
    ch.bind(event, (d: unknown) => saved.current(d));
    return () => {
      ch.unbind(event);
      client.unsubscribe(channel);
      client.disconnect();
    };
  }, [channel, event]);
}
```
------------------------------------------------------------------------
## STEP 5 : Consume in a Component
------------------------------------------------------------------------
```tsx
// src/app/chat/live.tsx
"use client";
import { useState } from "react";
import { useChannel } from "@/hooks/use-channel";

export function Live() {
  const [msgs, setMsgs] = useState<string[]>([]);
  useChannel("chat", "new-message", (d) =>
    setMsgs((m) => [...m, (d as { text: string }).text])
  );
  return <ul>{msgs.map((m, i) => <li key={i}>{m}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 6 : Private & Presence Channels
------------------------------------------------------------------------
Prefix with `private-` or `presence-` and add an auth endpoint.
```ts
// src/app/api/pusher/auth/route.ts
import { pusher } from "@/lib/pusher-server";

export async function POST(req: Request) {
  const body = new URLSearchParams(await req.text());
  const socketId = body.get("socket_id")!;
  const channel = body.get("channel_name")!;
  // authorize the current user here before signing
  const auth = pusher.authorizeChannel(socketId, channel);
  return Response.json(auth);
}
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   Browser A ──POST /api/message──► Next.js Route
                                        │ trigger()
                                        ▼
                                 Pusher Channels
                                   (hosted pub/sub)
                                        │ push
                    ┌───────────────────┼───────────────────┐
                    ▼                    ▼                   ▼
                Browser A            Browser B           Browser C
              (subscribed to "chat" via WebSocket)
```
Note (Windows): no native deps, so `pnpm` installs cleanly; if a
corporate proxy blocks WebSockets, pusher-js auto-falls back to
HTTP streaming.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep app id + secret server-only; expose just key and cluster
✓ Trigger events from server routes, never from the browser directly
✓ Use private-/presence- channels + auth for anything user-scoped
✓ Always unsubscribe and disconnect in the effect cleanup
✓ Namespace channels per resource (chat-{roomId}) to scope fan-out
✓ Keep event payloads small; send IDs and refetch heavy data
✓ Batch high-frequency events; respect the 10KB message limit
✓ Handle reconnection state in the UI (connecting / connected / failed)
```
