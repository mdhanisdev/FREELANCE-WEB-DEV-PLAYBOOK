# POSTHOG SETUP [ PRODUCT ANALYTICS + FEATURE FLAGS ]
------------------------------------------------------------------------
## STEP 1 : Install

PostHog gives events, funnels, session replay, and feature flags in one
platform. It works client-side and server-side.

```bash
pnpm add posthog-js posthog-node
```

```bash
# .env.local  (placeholders)
NEXT_PUBLIC_POSTHOG_KEY=phc_placeholder_key
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
```

------------------------------------------------------------------------
## STEP 2 : Client Provider

Initialise PostHog once and wrap the tree in the root layout.

```tsx
// app/providers.tsx
"use client";
import posthog from "posthog-js";
import { PostHogProvider } from "posthog-js/react";
import { useEffect } from "react";

export function PHProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
      capture_pageview: true,
      person_profiles: "identified_only", // no profiles for anon users
    });
  }, []);

  return <PostHogProvider client={posthog}>{children}</PostHogProvider>;
}
```

```tsx
// app/layout.tsx
import { PHProvider } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <PHProvider>{children}</PHProvider>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------
## STEP 3 : Capture Events

Name events `object_action`. Send only the properties you will analyse.

```tsx
"use client";
import { usePostHog } from "posthog-js/react";

export function UpgradeButton({ plan }: { plan: string }) {
  const posthog = usePostHog();
  return (
    <button
      onClick={() =>
        posthog.capture("plan_upgraded", { plan }) // no email/name
      }
    >
      Upgrade
    </button>
  );
}
```

------------------------------------------------------------------------
## STEP 4 : Identify Users (Minimal PII)

Identify with an opaque id so events tie to one person across sessions.
Do not stuff personal data into properties.

```ts
posthog.identify(session.userId, { plan: "pro" });
// On logout:
posthog.reset();
```

------------------------------------------------------------------------
## STEP 5 : Server-Side Capture

Use `posthog-node` for backend events (webhooks, cron, server actions).

```ts
// app/lib/posthog-server.ts
import { PostHog } from "posthog-node";

export const phServer = new PostHog(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
  host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
});
```

```ts
await phServer.capture({
  distinctId: userId,
  event: "invoice_paid",
  properties: { amount: 4900, currency: "usd" },
});
await phServer.shutdown(); // flush before the function exits
```

------------------------------------------------------------------------
## STEP 6 : Feature Flags

Flags let you roll out and A/B test without redeploying.

```text
  posthog.isFeatureEnabled("new-checkout")   // client boolean
  await phServer.isFeatureEnabled(            // server, per user
    "new-checkout", userId)
```

```text
  client capture --+
                   +--> PostHog --> insights / funnels / replay
  server capture --+                flags evaluated per distinctId
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep the project key + host in env vars
✓ Use person_profiles: identified_only to limit anon tracking
✓ Identify with an opaque id, never raw email as distinctId
✓ Name events object_action and keep properties lean
✓ Call shutdown()/flush() on server captures in serverless
✓ Gate risky features behind flags, not redeploys
✓ Never capture PII you will not actually query
✓ Turn on autocapture consciously — audit what it collects
```
