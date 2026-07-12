# VERCEL ANALYTICS SETUP [ TRAFFIC + WEB VITALS ]
------------------------------------------------------------------------
## STEP 1 : Install

Vercel Analytics gives privacy-friendly traffic data; Speed Insights
gives real-user Web Vitals. Both are drop-in for the App Router.

```bash
pnpm add @vercel/analytics @vercel/speed-insights
```

Enable both in the Vercel dashboard under your project's Analytics tab.

------------------------------------------------------------------------
## STEP 2 : Add to the Root Layout

Render both components once, near the end of `<body>`.

```tsx
// app/layout.tsx
import { Analytics } from "@vercel/analytics/react";
import { SpeedInsights } from "@vercel/speed-insights/next";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------
## STEP 3 : What Each Piece Measures

```text
  @vercel/analytics     -> page views, visitors, referrers, top paths
  @vercel/speed-insights-> LCP, INP, CLS, FCP, TTFB from real users

  visitor --> your page --> beacon --> Vercel dashboard
                             |
                   (no cookies, aggregated, privacy-first)
```

------------------------------------------------------------------------
## STEP 4 : Understand Web Vitals

Speed Insights reports Core Web Vitals from actual sessions, not lab
runs. Watch these thresholds:

```text
  LCP  Largest Contentful Paint   good < 2.5s
  INP  Interaction to Next Paint  good < 200ms
  CLS  Cumulative Layout Shift    good < 0.1
  FCP  First Contentful Paint     supporting metric
  TTFB Time To First Byte         supporting metric
```

------------------------------------------------------------------------
## STEP 5 : Custom Events (Analytics)

Track a few meaningful conversions. Keep properties non-personal.

```tsx
"use client";
import { track } from "@vercel/analytics";

export function CtaButton() {
  return (
    <button onClick={() => track("signup_clicked", { source: "hero" })}>
      Get started
    </button>
  );
}
```

Custom events require the Analytics Plus/Pro tier on Vercel.

------------------------------------------------------------------------
## STEP 6 : Local Development Notes

On Windows, beacons are suppressed in dev by default so you do not
pollute production data. Set debug mode to confirm wiring.

```tsx
<Analytics debug={process.env.NODE_ENV === "development"} />
```

Deploy to Vercel to see real numbers — data does not flow from a local
`pnpm dev` session.

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Add Analytics + SpeedInsights once, in the root layout
✓ Enable both products in the Vercel dashboard first
✓ Treat Web Vitals as a budget: LCP<2.5s, INP<200ms, CLS<0.1
✓ Keep custom-event names and props free of PII
✓ Track a handful of key conversions, not everything
✓ Verify with debug mode in dev; trust only deployed data
✓ No cookie banner needed — it is cookieless and aggregated
✓ Watch INP after interactive features ship
```
