# SENTRY SETUP [ ERROR MONITORING + PERFORMANCE ]
------------------------------------------------------------------------
## STEP 1 : Install via the Wizard

The wizard is the fastest path. It creates config files, wires the
Next.js plugin, and adds source-map upload for you.

```bash
pnpm dlx @sentry/wizard@latest -i nextjs
```

It generates:
- `sentry.client.config.ts`   (browser runtime)
- `sentry.server.config.ts`   (Node runtime)
- `sentry.edge.config.ts`     (edge runtime)
- `instrumentation.ts`        (registers server/edge configs)
- updates `next.config.ts`    (wraps with `withSentryConfig`)

If you prefer manual install: `pnpm add @sentry/nextjs`.

------------------------------------------------------------------------
## STEP 2 : Configure Sample Rates

Keep DSN in an env var. Never commit a real DSN — use a placeholder.

```bash
# .env.local  (Windows: edit with VS Code, do NOT commit)
NEXT_PUBLIC_SENTRY_DSN=https://examplekey@o0.ingest.sentry.io/0
SENTRY_AUTH_TOKEN=sntrys_placeholder_token
```

```ts
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  // Traces: sample low in prod to control cost.
  tracesSampleRate: process.env.NODE_ENV === "production" ? 0.1 : 1.0,
  // Session Replay: record a slice, but every session with an error.
  replaysSessionSampleRate: 0.05,
  replaysOnErrorSampleRate: 1.0,
  environment: process.env.NODE_ENV,
});
```

------------------------------------------------------------------------
## STEP 3 : Capture Exceptions Manually

Uncaught errors are auto-captured. Wrap risky calls to add context.

```ts
// app/lib/pay.ts
import * as Sentry from "@sentry/nextjs";

export async function charge(orderId: string) {
  try {
    return await provider.charge(orderId);
  } catch (err) {
    Sentry.captureException(err, {
      tags: { area: "billing" },
      extra: { orderId }, // safe id, NOT card numbers or emails
    });
    throw err;
  }
}
```

------------------------------------------------------------------------
## STEP 4 : Identify the User (Minimal PII)

Only attach an opaque id. Never send email, name, or address unless a
support workflow strictly requires it and consent exists.

```ts
import * as Sentry from "@sentry/nextjs";

Sentry.setUser({ id: session.userId }); // opaque id only
// On logout:
Sentry.setUser(null);
```

------------------------------------------------------------------------
## STEP 5 : Global Error Boundary

App Router needs a root `global-error.tsx` to catch render crashes.

```tsx
// app/global-error.tsx
"use client";
import * as Sentry from "@sentry/nextjs";
import { useEffect } from "react";

export default function GlobalError({
  error,
}: {
  error: Error & { digest?: string };
}) {
  useEffect(() => {
    Sentry.captureException(error);
  }, [error]);

  return (
    <html>
      <body>
        <h2>Something went wrong.</h2>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------
## STEP 6 : Source Maps + Releases

The wizard uploads source maps at build time using your auth token so
stack traces show original TypeScript, not minified output.

```text
  pnpm build
      |
      v
  withSentryConfig ---> uploads .map files ---> Sentry
      |                                            |
      v                                            v
  minified bundle shipped        readable stack traces in dashboard
```

Tie errors to a release so you know which deploy broke.

```bash
# CI or local build (Windows PowerShell / Git Bash)
pnpm dlx @sentry/cli releases new "$env:VERCEL_GIT_COMMIT_SHA"
pnpm dlx @sentry/cli releases finalize "$env:VERCEL_GIT_COMMIT_SHA"
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep DSN + auth token in env vars, never in git
✓ Lower tracesSampleRate in production to control quota
✓ Always ship source maps so traces are readable
✓ Attach only an opaque user id — no email, name, or PII
✓ Add a root global-error.tsx for render crashes
✓ Tag events by area (billing, auth) for fast triage
✓ Bind errors to a release/commit for regression tracking
✓ Scrub request bodies and headers before capture
✓ Alert on error-rate spikes, not on every single event
```
