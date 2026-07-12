# CHART-JS SETUP [ CHARTS ]
------------------------------------------------------------------------

Chart.js is a mature canvas-based charting library. It renders to a
single <canvas> (not SVG), which keeps the DOM light and performs well
with large datasets. `react-chartjs-2` provides the React bindings.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add chart.js react-chartjs-2
```

Chart.js v4 is tree-shakeable — you register only the pieces you use.

------------------------------------------------------------------------

## STEP 2 : Register Once

```ts
// lib/chartjs.ts
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  Tooltip,
  Legend,
} from "chart.js";

ChartJS.register(
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  Tooltip,
  Legend
);
```

Import this module once (from your chart component) to register the
controllers, scales, and plugins globally.

------------------------------------------------------------------------

## STEP 3 : Render Pipeline

```text
+------------------------------------------------------------+
|  data{labels,datasets}  +  options                         |
|              \              /                              |
|               v            v                               |
|            <Bar data options />                            |
|                     |                                      |
|                     v                                      |
|          Chart.js core -> <canvas> (GPU-friendly)          |
|                     |                                      |
|            tooltips / legend / scales plugins              |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 4 : A Line Chart Component

```tsx
// components/charts/sales-line.tsx
"use client";
import "@/lib/chartjs";
import { Line } from "react-chartjs-2";
import type { ChartData, ChartOptions } from "chart.js";

const data: ChartData<"line"> = {
  labels: ["Jan", "Feb", "Mar", "Apr"],
  datasets: [
    {
      label: "Sales",
      data: [120, 190, 140, 220],
      borderColor: "#2563eb",
      backgroundColor: "rgba(37,99,235,0.2)",
      tension: 0.3,
    },
  ],
};

const options: ChartOptions<"line"> = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: { legend: { position: "bottom" } },
};

export function SalesLine() {
  return (
    <div className="relative h-64 w-full">
      <Line data={data} options={options} />
    </div>
  );
}
```

`maintainAspectRatio: false` plus a sized wrapper is how you control
height — the canvas fills its relative parent.

------------------------------------------------------------------------

## STEP 5 : Use in a Page

```tsx
// app/sales/page.tsx
import { SalesLine } from "@/components/charts/sales-line";

export default function Page() {
  return (
    <div className="rounded-lg border p-4">
      <SalesLine />
    </div>
  );
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

Canvas rendering is handled by the browser, so there are no native
dependencies to compile on Windows. Behavior is identical across all
platforms; just ensure the component is `"use client"`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Register only the scales/elements you use for smaller bundles
✓ Put registration in one lib module, import it from charts
✓ Wrap the canvas in a sized relative div for responsive height
✓ Set maintainAspectRatio:false to control height with CSS
✓ Type data/options as ChartData<"line"> / ChartOptions<"line">
✓ Keep chart components "use client" — canvas needs the browser
✓ Reuse dataset color tokens instead of inline hex per chart
✓ Destroy/recreate is automatic in react-chartjs-2 — avoid manual refs
```
