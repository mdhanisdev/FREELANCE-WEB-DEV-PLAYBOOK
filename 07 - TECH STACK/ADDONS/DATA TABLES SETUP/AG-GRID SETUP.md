# AG-GRID SETUP [ DATA TABLES ]
------------------------------------------------------------------------

AG Grid is a feature-complete enterprise data grid: virtualized rows,
built-in sorting/filtering, cell editing, grouping and pivoting. Reach
for it when you need a spreadsheet-grade grid out of the box rather than
a headless toolkit.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add ag-grid-react ag-grid-community
```

Community is MIT and free. Enterprise features (pivoting, row grouping
UI, Excel export) require `ag-grid-enterprise` + a license key.

------------------------------------------------------------------------

## STEP 2 : How It Renders

```text
+-------------------------------------------------------------+
|  rowData[]   colDefs[]                                      |
|      \          /                                           |
|       v        v                                            |
|   <AgGridReact/>  --->  virtualized viewport                |
|                          |  only visible rows in DOM        |
|                          |  scroll -> recycle row nodes     |
|                          v                                  |
|                     header | body | pinned | pagination     |
+-------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Client Grid Component

AG Grid touches the DOM directly, so it must be a Client Component and
should be dynamically imported to keep it out of the server bundle.

```tsx
// components/grid/invoice-grid.tsx
"use client";
import { AgGridReact } from "ag-grid-react";
import { ColDef } from "ag-grid-community";
import { useMemo, useState } from "react";
import "ag-grid-community/styles/ag-grid.css";
import "ag-grid-community/styles/ag-theme-quartz.css";

type Invoice = { customer: string; status: string; amount: number };

export default function InvoiceGrid({ rows }: { rows: Invoice[] }) {
  const [colDefs] = useState<ColDef<Invoice>[]>([
    { field: "customer", filter: true, flex: 1 },
    { field: "status", filter: true },
    {
      field: "amount",
      valueFormatter: (p) =>
        new Intl.NumberFormat("en-US", {
          style: "currency",
          currency: "USD",
        }).format(p.value),
    },
  ]);

  const defaultColDef = useMemo<ColDef>(
    () => ({ sortable: true, resizable: true }),
    []
  );

  return (
    <div className="ag-theme-quartz" style={{ height: 480, width: "100%" }}>
      <AgGridReact
        rowData={rows}
        columnDefs={colDefs}
        defaultColDef={defaultColDef}
        pagination
        paginationPageSize={20}
      />
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Lazy Load in a Page

```tsx
// app/grid/page.tsx
import dynamic from "next/dynamic";

const InvoiceGrid = dynamic(
  () => import("@/components/grid/invoice-grid"),
  { ssr: false, loading: () => <p>Loading grid…</p> }
);

export default function Page() {
  const rows = [
    { customer: "Acme", status: "paid", amount: 1200 },
    { customer: "Globex", status: "pending", amount: 840 },
  ];
  return <InvoiceGrid rows={rows} />;
}
```

------------------------------------------------------------------------

## STEP 5 : Theming

Quartz is the modern default. Override CSS variables in globals.css:

```json
{
  "--ag-header-height": "40px",
  "--ag-row-height": "36px",
  "--ag-accent-color": "#2563eb"
}
```

Apply them under the `.ag-theme-quartz` selector in your stylesheet.

------------------------------------------------------------------------

## WINDOWS NOTES

Pure JS — no native build step. If you see missing grid styles on
Windows dev servers, confirm both CSS imports resolved and that the
container has an explicit height (grid collapses to 0 otherwise).

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Import AG Grid with ssr:false — it needs the browser DOM
✓ Always give the wrapper a fixed height or the grid won't show
✓ Memoize defaultColDef / colDefs to prevent full re-renders
✓ Use valueFormatter for display, keep raw values for sorting
✓ Prefer server-side row model for very large datasets
✓ Only add ag-grid-enterprise when you truly need its features
✓ Set a license key once at app bootstrap, never per component
✓ Reuse one theme via CSS variables for visual consistency
```
