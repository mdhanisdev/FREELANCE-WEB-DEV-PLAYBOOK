# RECHARTS SETUP [ CHARTS ]
------------------------------------------------------------------------

Recharts is a composable charting library built on React + D3. It is
the engine behind shadcn/ui charts, so it is the default choice for an
App Router dashboard: declarative components, responsive containers,
and theme-able via CSS variables.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add recharts
pnpm dlx shadcn@latest add chart
```

The `chart` component adds `components/ui/chart.tsx` with
`ChartContainer`, `ChartTooltip`, and CSS-variable theming.

------------------------------------------------------------------------

## STEP 2 : Composition Model

```text
+------------------------------------------------------------+
|  ResponsiveContainer                                       |
|    +----------------------------------------------------+  |
|    |  <BarChart data=[]>                                |  |
|    |     <CartesianGrid/>                               |  |
|    |     <XAxis/>  <YAxis/>                             |  |
|    |     <ChartTooltip/>                                |  |
|    |     <Bar dataKey="value" fill="var(--chart-1)"/>   |  |
|    |  </BarChart>                                       |  |
|    +----------------------------------------------------+  |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Chart Config (shadcn)

```ts
// components/charts/config.ts
import type { ChartConfig } from "@/components/ui/chart";

export const revenueConfig = {
  desktop: { label: "Desktop", color: "var(--chart-1)" },
  mobile: { label: "Mobile", color: "var(--chart-2)" },
} satisfies ChartConfig;
```

------------------------------------------------------------------------

## STEP 4 : A Bar Chart

```tsx
// components/charts/revenue-bar.tsx
"use client";
import { Bar, BarChart, CartesianGrid, XAxis } from "recharts";
import {
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
} from "@/components/ui/chart";
import { revenueConfig } from "./config";

const data = [
  { month: "Jan", desktop: 186, mobile: 80 },
  { month: "Feb", desktop: 305, mobile: 200 },
  { month: "Mar", desktop: 237, mobile: 120 },
];

export function RevenueBar() {
  return (
    <ChartContainer config={revenueConfig} className="h-64 w-full">
      <BarChart data={data}>
        <CartesianGrid vertical={false} />
        <XAxis dataKey="month" tickLine={false} axisLine={false} />
        <ChartTooltip content={<ChartTooltipContent />} />
        <Bar dataKey="desktop" fill="var(--color-desktop)" radius={4} />
        <Bar dataKey="mobile" fill="var(--color-mobile)" radius={4} />
      </BarChart>
    </ChartContainer>
  );
}
```

`ChartContainer` maps each config key to a `--color-<key>` variable, so
`fill="var(--color-desktop)"` resolves to your theme color.

------------------------------------------------------------------------

## STEP 5 : Drop Into a Page

```tsx
// app/dashboard/page.tsx
import { RevenueBar } from "@/components/charts/revenue-bar";

export default function Page() {
  return (
    <div className="rounded-lg border p-4">
      <h2 className="mb-4 text-lg font-semibold">Revenue</h2>
      <RevenueBar />
    </div>
  );
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

Recharts is pure JS — no native deps. Charts must be Client Components
(`"use client"`) because they measure the DOM; the dev experience is
identical across Windows, macOS, and Linux.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use shadcn's ChartContainer for theme-aware CSS-variable colors
✓ Always wrap charts in ResponsiveContainer / ChartContainer
✓ Give the container an explicit height class (e.g. h-64)
✓ Keep chart components "use client"; pass data from the server
✓ Define ChartConfig with `satisfies` for autocomplete + safety
✓ Disable axis lines/ticks for a cleaner dashboard look
✓ Reuse one config object across related charts
✓ Prefer var(--chart-N) tokens over hard-coded hex values
```
