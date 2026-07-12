# FLAGSMITH SETUP [ FEATURE FLAGS ]
------------------------------------------------------------------------
Flagsmith is the alternative when you want a dedicated, open-source flag
platform you can self-host: environments, segments, and remote config.
This guide wires the JS + Node SDKs into Next.js App Router with
TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add flagsmith flagsmith-nodejs
```
------------------------------------------------------------------------
## STEP 2 : Configure Environment

```bash
# .env.local
NEXT_PUBLIC_FLAGSMITH_ENV_ID=your_client_side_env_key
FLAGSMITH_SERVER_KEY=ser.your_server_side_key
```

```text
Key types
  ┌───────────────┬───────────────────────────────┐
  │ env id        │ client-side, browser-safe      │
  │ ser.* key     │ server-side, keep secret       │
  └───────────────┴───────────────────────────────┘
```
------------------------------------------------------------------------
## STEP 3 : Server-Side Evaluation

```ts
// lib/flagsmith-server.ts
import Flagsmith from "flagsmith-nodejs";

export const flagsmith = new Flagsmith({
  environmentKey: process.env.FLAGSMITH_SERVER_KEY!,
});

export async function isEnabled(name: string, identity?: string) {
  const flags = identity
    ? await flagsmith.getIdentityFlags(identity)
    : await flagsmith.getEnvironmentFlags();
  return flags.isFeatureEnabled(name);
}
```
------------------------------------------------------------------------
## STEP 4 : Gate a Server Component

```tsx
// app/dashboard/page.tsx
import { isEnabled } from "@/lib/flagsmith-server";

export default async function Dashboard() {
  const on = await isEnabled("new_dashboard", "user-123");
  return on ? <NewDashboard /> : <LegacyDashboard />;
}
```
------------------------------------------------------------------------
## STEP 5 : Client Provider + Hook

```tsx
// components/flagsmith-provider.tsx
"use client";

import { FlagsmithProvider } from "flagsmith/react";
import { createFlagsmithInstance } from "flagsmith/isomorphic";
import { useRef } from "react";

export function Providers({ children }: { children: React.ReactNode }) {
  const instance = useRef(createFlagsmithInstance());
  return (
    <FlagsmithProvider
      flagsmith={instance.current}
      options={{ environmentID: process.env.NEXT_PUBLIC_FLAGSMITH_ENV_ID! }}
    >
      {children}
    </FlagsmithProvider>
  );
}
```

```tsx
// usage
"use client";
import { useFlags } from "flagsmith/react";

export function Beta() {
  const flags = useFlags(["new_dashboard"]);
  return flags.new_dashboard.enabled ? <span>Beta on</span> : null;
}
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# toggle the flag in the Flagsmith dashboard → refresh
```

Windows note: if self-hosting Flagsmith via Docker Desktop on Windows,
point the SDK `apiUrl` at `http://localhost:8000/api/v1/`; SaaS needs no
apiUrl override.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use identity flags for per-user targeting and segments
✓ Keep ser.* server keys secret; only env id in the browser
✓ Evaluate blocking flags server-side to prevent flicker
✓ Cache environment flags briefly to cut API round-trips
✓ Use remote config values, not just boolean on/off
✓ Model rollout stages with environments (dev/stage/prod)
✓ Archive stale flags after a full rollout
✓ Self-host behind your own TLS if data residency matters
```
