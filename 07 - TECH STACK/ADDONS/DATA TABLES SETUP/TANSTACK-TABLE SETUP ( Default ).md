# TANSTACK TABLE SETUP [ DATA TABLES ]
------------------------------------------------------------------------

TanStack Table (v8) is a headless table library. It owns the logic
(sorting, filtering, pagination, row selection) and leaves the markup
to you, so it pairs perfectly with shadcn/ui + Tailwind.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @tanstack/react-table
pnpm add @tanstack/match-sorter-utils   # optional fuzzy filtering
```

Headless means zero styles ship. You render `<table>` yourself.

------------------------------------------------------------------------

## STEP 2 : Architecture

```text
+-----------------------------------------------------------+
|  data[]  +  columns[]  ->  useReactTable(options)         |
|                                   |                       |
|                                   v                        |
|            table instance (getHeaderGroups / getRowModel)  |
|                                   |                        |
|            +----------------------+---------------------+  |
|            v                      v                     v  |
|         sorting              filtering            pagination|
|            |                      |                     |  |
|            +----------> <table> (you render) <----------+  |
+-----------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Define Columns

```ts
// components/table/columns.ts
import { ColumnDef } from "@tanstack/react-table";

export type Invoice = {
  id: string;
  customer: string;
  status: "paid" | "pending" | "void";
  amount: number;
};

export const columns: ColumnDef<Invoice>[] = [
  { accessorKey: "customer", header: "Customer" },
  { accessorKey: "status", header: "Status" },
  {
    accessorKey: "amount",
    header: () => <div className="text-right">Amount</div>,
    cell: ({ row }) =>
      new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
      }).format(row.getValue("amount")),
  },
];
```

------------------------------------------------------------------------

## STEP 4 : The DataTable Component

```tsx
// components/table/data-table.tsx
"use client";
import {
  flexRender,
  getCoreRowModel,
  getSortedRowModel,
  getPaginationRowModel,
  useReactTable,
  type ColumnDef,
  type SortingState,
} from "@tanstack/react-table";
import { useState } from "react";

export function DataTable<TData, TValue>({
  columns,
  data,
}: {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
}) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const table = useReactTable({
    data,
    columns,
    state: { sorting },
    onSortingChange: setSorting,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
  });

  return (
    <table className="w-full text-sm">
      <thead>
        {table.getHeaderGroups().map((hg) => (
          <tr key={hg.id}>
            {hg.headers.map((h) => (
              <th
                key={h.id}
                onClick={h.column.getToggleSortingHandler()}
                className="cursor-pointer px-3 py-2 text-left"
              >
                {flexRender(h.column.columnDef.header, h.getContext())}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id} className="border-t">
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id} className="px-3 py-2">
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Use It (Server Component fetch)

```tsx
// app/invoices/page.tsx
import { DataTable } from "@/components/table/data-table";
import { columns } from "@/components/table/columns";

export default async function Page() {
  const data = await fetch("https://api.example.com/invoices", {
    cache: "no-store",
  }).then((r) => r.json());
  return <DataTable columns={columns} data={data} />;
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

Nothing native to compile. Use `pnpm` in PowerShell or Git Bash;
paths with spaces (e.g. Desktop folders) work — keep imports on the
`@/` alias to avoid backslash issues.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep columns[] in its own file — reuse across tables
✓ Memoize data and columns to avoid re-instantiating the table
✓ Mark the DataTable "use client"; fetch in a Server Component
✓ Type ColumnDef<T> so cell renderers stay type-safe
✓ Enable only the row models you use (tree-shakeable)
✓ Debounce global filter input on large datasets
✓ Use getRowId for stable keys with server pagination
✓ Persist sorting/pagination in the URL via searchParams
```
