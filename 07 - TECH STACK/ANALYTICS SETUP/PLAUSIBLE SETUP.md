# PLAUSIBLE SETUP [ PRIVACY-FRIENDLY ANALYTICS ]
------------------------------------------------------------------------
## STEP 1 : When To Choose Plausible

Plausible is lightweight (~1KB), cookieless, and GDPR/CCPA-friendly by
design. It collects no personal data and needs no cookie banner —
ideal when privacy and page speed matter more than deep funnels.

```text
  Google Analytics -> deep, free, but cookies + consent banner needed
  PostHog          -> product analytics, funnels, flags, replay
  Plausible        -> simple traffic stats, no cookies, no banner
  Choose Plausible for clean, compliant, minimal analytics.
```

------------------------------------------------------------------------
## STEP 2 : Add the Script

Use Next.js `<Script>` with your verified domain. Put it in the root
layout so it loads on every route.

```tsx
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        <Script
          defer
          data-domain="example.com"
          src="https://plausible.io/js/script.js"
          strategy="afterInteractive"
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

------------------------------------------------------------------------
## STEP 3 : Enable Custom Events

Swap the script for the `.tagged-events` build to send custom goals.

```tsx
<Script
  defer
  data-domain="example.com"
  src="https://plausible.io/js/script.tagged-events.js"
  strategy="afterInteractive"
/>
```

------------------------------------------------------------------------
## STEP 4 : Fire Custom Events

The script exposes `window.plausible`. Add a tiny typed wrapper.

```ts
// app/lib/plausible.ts
declare global {
  interface Window {
    plausible?: (
      event: string,
      opts?: { props?: Record<string, string | number> }
    ) => void;
  }
}

export function trackEvent(
  name: string,
  props?: Record<string, string | number>
) {
  window.plausible?.(name, props ? { props } : undefined);
}
```

```tsx
"use client";
import { trackEvent } from "@/app/lib/plausible";

export function SignupButton() {
  return (
    <button onClick={() => trackEvent("Signup", { plan: "pro" })}>
      Sign up
    </button>
  );
}
```

Register each event name as a Goal in the Plausible dashboard.

------------------------------------------------------------------------
## STEP 5 : How It Stays Private

```text
  visitor --> script.js --> Plausible
                 |
          no cookies, no cross-site id,
          no persistent fingerprint
                 |
        aggregated stats only --> dashboard
```

Because nothing personal is stored, no consent banner is required in
cookie-regulated regions — a key operational win.

------------------------------------------------------------------------
## STEP 6 : Optional — Proxy to Avoid Blockers

Route the script through your own domain via Next.js rewrites so ad
blockers do not strip it.

```ts
// next.config.ts
export default {
  async rewrites() {
    return [
      { source: "/stats/js/script.js", destination: "https://plausible.io/js/script.js" },
      { source: "/stats/api/event", destination: "https://plausible.io/api/event" },
    ];
  },
};
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Choose Plausible when privacy + light weight beat deep funnels
✓ No cookie banner needed — it is cookieless by design
✓ Load the script afterInteractive to protect page speed
✓ Use the tagged-events build only when you need custom goals
✓ Register each custom event as a Goal in the dashboard
✓ Keep event props non-personal — never send PII
✓ Proxy via rewrites if ad blockers are hurting coverage
✓ Verify data-domain matches your Plausible site exactly
```
