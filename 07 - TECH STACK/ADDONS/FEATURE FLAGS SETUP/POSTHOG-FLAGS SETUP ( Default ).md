# POSTHOG-FLAGS SETUP [ FEATURE FLAGS ]
------------------------------------------------------------------------
PostHog is our default feature-flag platform: flags, A/B experiments, and
analytics in one tool with a generous free tier. This guide evaluates
flags on the server for App Router pages and on the client for
interactivity, using TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add posthog-node posthog-js
```

`posthog-node` runs on the server; `posthog-js` runs in the browser.
------------------------------------------------------------------------
## STEP 2 : Configure Environment

```bash
# .env.local
NEXT_PUBLIC_POSTHOG_KEY=phc_your_project_key
NEXT_PUBLIC_POSTHOG_HOST=https://eu.i.posthog.com
POSTHOG_PERSONAL_API_KEY=phx_your_personal_key
```

```text
Evaluation surfaces
  ┌─────────────┬──────────────────────────────┐
  │ server      │ posthog-node → RSC / route    │
  │ client      │ posthog-js  → useFeatureFlag  │
  └─────────────┴──────────────────────────────┘
```
------------------------------------------------------------------------
## STEP 3 : Server-Side Flag Client

```ts
// lib/posthog-server.ts
import { PostHog } from "posthog-node";

export function getPostHog() {
  return new PostHog(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
    host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
    flushAt: 1,
    flushInterval: 0,
  });
}
```
------------------------------------------------------------------------
## STEP 4 : Evaluate a Flag in a Server Component

```tsx
// app/dashboard/page.tsx
import { getPostHog } from "@/lib/posthog-server";

export default async function Dashboard() {
  const ph = getPostHog();
  const enabled = await ph.isFeatureEnabled("new-dashboard", "user-123");
  await ph.shutdown(); // flush before the request ends

  return enabled ? <NewDashboard /> : <LegacyDashboard />;
}
```

Always `shutdown()` (or `flush()`) in serverless so events are not lost.
------------------------------------------------------------------------
## STEP 5 : Client Provider + Hook

```tsx
// components/posthog-provider.tsx
"use client";

import posthog from "posthog-js";
import { PostHogProvider } from "posthog-js/react";
import { useEffect } from "react";

export function Providers({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
      capture_pageview: false,
    });
  }, []);
  return <PostHogProvider client={posthog}>{children}</PostHogProvider>;
}
```

```tsx
// usage
"use client";
import { useFeatureFlagEnabled } from "posthog-js/react";

export function Beta() {
  const on = useFeatureFlagEnabled("new-dashboard");
  return on ? <span>Beta on</span> : null;
}
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# toggle the flag in PostHog → refresh the page
```

Windows note: nothing OS-specific here, but restart `pnpm dev` after
editing `.env.local` since Next reads env only at boot.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Evaluate gating flags on the server to avoid UI flicker
✓ Always shutdown()/flush() posthog-node in serverless
✓ Pass a stable distinct_id so rollouts are deterministic
✓ Use bootstrap data to hydrate client flags without a round-trip
✓ Keep personal API keys server-side, never NEXT_PUBLIC
✓ Name flags by intent (new-dashboard) not by ticket id
✓ Clean up stale flags once a rollout hits 100%
✓ Pair flags with experiments to measure impact, not just ship
```
