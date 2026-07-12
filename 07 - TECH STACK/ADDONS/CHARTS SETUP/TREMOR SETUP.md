# TREMOR SETUP [ CHARTS ]
------------------------------------------------------------------------

Tremor is a set of React components for dashboards — charts, KPI cards,
bars, and trackers — styled with Tailwind. It gives you polished,
opinionated blocks fast, ideal when you want a dashboard shipped rather
than assembled from primitives.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @tremor/react
pnpm add -D tailwindcss @tailwindcss/forms
```

Tremor builds on Tailwind, so your project must already have Tailwind
configured (the default for a shadcn App Router setup).

------------------------------------------------------------------------

## STEP 2 : Tailwind Content Path

Tremor ships class names you must include in Tailwind scanning:

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";

export default {
  content: [
    "./app/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./node_modules/@tremor/**/*.{js,ts,jsx,tsx}",
  ],
} satisfies Config;
```

------------------------------------------------------------------------

## STEP 3 : Component Map

```text
+------------------------------------------------------------+
|  <Card>                                                    |
|    <Title>  <Text>            <- KPI header                |
|    <Metric>2,340</Metric>                                  |
|    +----------------------------------------------------+  |
|    | <AreaChart data categories index />               |  |
|    +----------------------------------------------------+  |
|    <BarList data /> <ProgressBar /> <Tracker />           |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 4 : KPI Card + Area Chart

```tsx
// components/charts/traffic-card.tsx
"use client";
import { Card, Title, AreaChart } from "@tremor/react";

const data = [
  { date: "Jan", Visitors: 2890 },
  { date: "Feb", Visitors: 3120 },
  { date: "Mar", Visitors: 2760 },
];

export function TrafficCard() {
  return (
    <Card className="max-w-lg">
      <Title>Site Visitors</Title>
      <AreaChart
        className="mt-4 h-56"
        data={data}
        index="date"
        categories={["Visitors"]}
        colors={["blue"]}
        valueFormatter={(n) => n.toLocaleString("en-US")}
        showLegend={false}
      />
    </Card>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Use in a Server Page

```tsx
// app/analytics/page.tsx
import { TrafficCard } from "@/components/charts/traffic-card";

export default function Page() {
  return (
    <main className="grid gap-4 p-6 md:grid-cols-2">
      <TrafficCard />
    </main>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Dark Mode

Tremor respects the `dark` class. If you use shadcn's theme provider,
Tremor components adopt dark styling automatically — no extra config.

------------------------------------------------------------------------

## WINDOWS NOTES

No native modules. The only common cross-platform gotcha is the
Tailwind `content` glob: keep forward slashes in the config even on
Windows — Tailwind normalizes them internally.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Add the @tremor/** path to Tailwind content or styles vanish
✓ Mark chart components "use client"; fetch data server-side
✓ Use valueFormatter with toLocaleString for currency/number labels
✓ Set an explicit chart height class (e.g. h-56)
✓ Stick to Tremor color names for consistent palettes
✓ Compose KPI Cards from Title + Metric + chart, not custom divs
✓ Verify Tremor's Tailwind version matches your project's
✓ Prefer Tremor for full dashboards, Recharts for bespoke charts
```
