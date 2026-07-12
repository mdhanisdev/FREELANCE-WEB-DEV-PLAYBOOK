# SUPABASE-REALTIME SETUP [ REALTIME ]
------------------------------------------------------------------------
Supabase Realtime pushes Postgres changes, broadcast messages, and
presence over WebSockets. If you already use Supabase for data, realtime
is free of extra infra: listen to table changes or broadcast ephemeral
events on named channels.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add @supabase/supabase-js @supabase/ssr
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
```text
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
```
The anon key is safe for the browser only because Row Level Security
gates every table. Enable RLS before exposing any data.
------------------------------------------------------------------------
## STEP 3 : Browser Client
------------------------------------------------------------------------
```ts
// src/lib/supabase-client.ts
import { createBrowserClient } from "@supabase/ssr";

export const supabase = createBrowserClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```
------------------------------------------------------------------------
## STEP 4 : Enable Realtime on a Table
------------------------------------------------------------------------
In the SQL editor, add the table to the realtime publication and turn on
RLS with a read policy.
```json
{
  "sql": [
    "alter publication supabase_realtime add table public.messages;",
    "alter table public.messages enable row level security;",
    "create policy read_all on public.messages for select using (true);"
  ]
}
```
------------------------------------------------------------------------
## STEP 5 : Subscribe to Postgres Changes
------------------------------------------------------------------------
```tsx
// src/app/chat/live.tsx
"use client";
import { useEffect, useState } from "react";
import { supabase } from "@/lib/supabase-client";

export function Live() {
  const [msgs, setMsgs] = useState<Array<{ id: number; text: string }>>([]);
  useEffect(() => {
    const channel = supabase
      .channel("room-1")
      .on(
        "postgres_changes",
        { event: "INSERT", schema: "public", table: "messages" },
        (payload) => setMsgs((m) => [...m, payload.new as { id: number; text: string }])
      )
      .subscribe();
    return () => {
      supabase.removeChannel(channel);
    };
  }, []);
  return <ul>{msgs.map((m) => <li key={m.id}>{m.text}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 6 : Broadcast & Presence
------------------------------------------------------------------------
```ts
// ephemeral events that never touch the database
const channel = supabase.channel("cursor", { config: { presence: { key: userId } } });
channel
  .on("broadcast", { event: "move" }, ({ payload }) => render(payload))
  .on("presence", { event: "sync" }, () => console.log(channel.presenceState()))
  .subscribe(async (status) => {
    if (status === "SUBSCRIBED") await channel.track({ online: true });
  });
channel.send({ type: "broadcast", event: "move", payload: { x: 10, y: 20 } });
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   INSERT ──► Postgres ──► WAL ──► Realtime server
                                       │ (RLS filtered)
                                       ▼
                               WebSocket fan-out
                    ┌──────────────────┼──────────────────┐
                    ▼                   ▼                  ▼
                Browser A           Browser B          Browser C
   Broadcast/Presence flow entirely in-memory, bypassing the database.
```
Note (Windows): the JS SDK has no native deps; `supabase start` for a
local stack needs Docker Desktop running under WSL2.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Enable RLS on every realtime table before exposing the anon key
✓ postgres_changes respects RLS; users only receive rows they can read
✓ Use broadcast/presence for ephemeral data (cursors, typing) not the DB
✓ Always removeChannel in cleanup to avoid leaking subscriptions
✓ Scope channels per room/resource to limit fan-out volume
✓ Keep realtime-enabled tables lean; large row payloads slow delivery
✓ Add filters (filter: "room_id=eq.1") to reduce client-side noise
✓ Monitor concurrent connection limits on your plan
```
