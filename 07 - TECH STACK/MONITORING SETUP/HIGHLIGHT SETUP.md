# HIGHLIGHT SETUP [ SESSION REPLAY + ERROR MONITORING ]
------------------------------------------------------------------------
## STEP 1 : When To Choose Highlight

Highlight.io combines session replay, error monitoring, and logs in one
tool. Reach for it when reproducing a bug matters as much as the stack
trace — you get a video-like replay tied to each error.

```text
  Sentry   -> best-in-class errors + performance, no full replay focus
  Highlight-> replay-first: "watch what the user did before the crash"
  Use Highlight when support/QA need to SEE the broken session.
```

------------------------------------------------------------------------
## STEP 2 : Install

```bash
pnpm add @highlight-run/next highlight.run
```

Put your project id in an env var (public — it is safe client-side).

```bash
# .env.local
NEXT_PUBLIC_HIGHLIGHT_PROJECT_ID=example_project_id
```

------------------------------------------------------------------------
## STEP 3 : Initialise the Client Provider

Wrap the App Router tree with the provider in the root layout.

```tsx
// app/providers.tsx
"use client";
import { HighlightInit } from "@highlight-run/next/client";

export function HighlightProvider() {
  return (
    <HighlightInit
      projectId={process.env.NEXT_PUBLIC_HIGHLIGHT_PROJECT_ID}
      serviceName="my-next-app"
      tracingOrigins
      networkRecording={{
        enabled: true,
        recordHeadersAndBody: true,
        // Never record sensitive fields:
        urlBlocklist: ["/api/auth", "/api/billing"],
      }}
    />
  );
}
```

```tsx
// app/layout.tsx
import { HighlightProvider } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <HighlightProvider />
        {children}
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------
## STEP 4 : Mask PII in the Replay

Replay can record real inputs. Mask anything sensitive so PII never
leaves the browser.

```tsx
// Add the highlight-mask class to any sensitive element
<input className="highlight-mask" name="email" />
<div className="highlight-mask">{user.fullName}</div>
```

------------------------------------------------------------------------
## STEP 5 : Report Errors From the Server

Wrap Next.js server error handling with the Highlight helper.

```ts
// app/lib/highlight.ts
import { Highlight } from "@highlight-run/next/server";

export const withHighlight = Highlight({
  projectID: process.env.NEXT_PUBLIC_HIGHLIGHT_PROJECT_ID!,
});
```

```tsx
// app/error.tsx
"use client";
import { useEffect } from "react";
import { H } from "highlight.run";

export default function Error({ error }: { error: Error }) {
  useEffect(() => {
    H.consumeError(error);
  }, [error]);
  return <h2>Something went wrong.</h2>;
}
```

------------------------------------------------------------------------
## STEP 6 : Link a Session to a User

```ts
import { H } from "highlight.run";

// Opaque id only; add email only with explicit consent
H.identify(session.userId, {});
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Choose Highlight when replay is the killer feature you need
✓ Mask every sensitive field with highlight-mask
✓ Blocklist auth and billing URLs from network recording
✓ Identify sessions with an opaque id, not email by default
✓ Keep session sample rate modest to control storage cost
✓ Pair replays with error consumeError for full context
✓ Review recorded network bodies for accidental PII leaks
✓ Set a clear data-retention window in project settings
```
