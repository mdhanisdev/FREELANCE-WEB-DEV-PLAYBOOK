# ABLY SETUP [ REALTIME ]
------------------------------------------------------------------------
Ably is a hosted realtime platform with pub/sub channels, guaranteed
message ordering, history/rewind, and presence. Clients authenticate via
short-lived token requests minted by your server, never a raw API key.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add ably
pnpm add @ably/react
```
`ably` is the isomorphic SDK; `@ably/react` adds hooks and providers.
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create an app at ably.com and copy the root API key.
```text
ABLY_API_KEY=xxxxxx.yyyyyy:zzzzzzzzzzzzzzzz
```
The API key is server-only. Browsers receive short-lived tokens instead.
------------------------------------------------------------------------
## STEP 3 : Token Auth Endpoint
------------------------------------------------------------------------
```ts
// src/app/api/ably/token/route.ts
import { NextResponse } from "next/server";
import Ably from "ably";

export async function GET() {
  const rest = new Ably.Rest(process.env.ABLY_API_KEY!);
  const tokenRequest = await rest.auth.createTokenRequest({
    clientId: "web-user", // set to the authenticated user id
  });
  return NextResponse.json(tokenRequest);
}
```
------------------------------------------------------------------------
## STEP 4 : Client Provider
------------------------------------------------------------------------
```tsx
// src/app/providers.tsx
"use client";
import * as Ably from "ably";
import { AblyProvider, ChannelProvider } from "@ably/react";

const client = new Ably.Realtime({ authUrl: "/api/ably/token" });

export function RealtimeProvider({ children }: { children: React.ReactNode }) {
  return (
    <AblyProvider client={client}>
      <ChannelProvider channelName="chat">{children}</ChannelProvider>
    </AblyProvider>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Subscribe & Publish
------------------------------------------------------------------------
```tsx
// src/app/chat/live.tsx
"use client";
import { useState } from "react";
import { useChannel } from "@ably/react";

export function Live() {
  const [msgs, setMsgs] = useState<string[]>([]);
  const { channel } = useChannel("chat", (m) =>
    setMsgs((prev) => [...prev, m.data.text])
  );
  return (
    <>
      <button onClick={() => channel.publish("msg", { text: "hi" })}>Send</button>
      <ul>{msgs.map((m, i) => <li key={i}>{m}</li>)}</ul>
    </>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Server Publish
------------------------------------------------------------------------
```ts
// src/app/api/broadcast/route.ts
import Ably from "ably";

export async function POST(req: Request) {
  const { text } = await req.json();
  const rest = new Ably.Rest(process.env.ABLY_API_KEY!);
  await rest.channels.get("chat").publish("msg", { text });
  return Response.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   Browser ──GET /api/ably/token──► Next.js ──signs token──► Browser
        │                                                       │
        └────────── Realtime WebSocket (token auth) ────────────┘
                                 │
                          Ably Global Edge
                          (ordering + history)
                                 │
        Server ──REST publish──► channel "chat" ──► all subscribers
```
Note (Windows): the SDK is pure JS with no native build step, so
installs are clean; token auth avoids storing the key in the client.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Never ship the API key to the browser; always use token auth
✓ Set clientId per user so presence and rate limits are per-identity
✓ Scope capabilities in token requests (subscribe-only where possible)
✓ Use channel history/rewind to backfill missed messages on reconnect
✓ Keep one Realtime client instance; share it via a provider
✓ Namespace channels per resource to bound fan-out and cost
✓ Handle connection state changes (suspended/failed) in the UI
✓ Keep payloads small; publish IDs and fetch heavy data over HTTP
```
