# TANSTACK-VIRTUAL SETUP [ VIRTUALIZATION ]
------------------------------------------------------------------------
TanStack Virtual is a headless virtualization hook for rendering huge
lists, grids, and tables while keeping the DOM tiny. It is framework-
agnostic, TypeScript-first, and unopinionated about markup — the default
for the stack because it supports dynamic measurement and Tailwind out
of the box.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add @tanstack/react-virtual
```
Windows note: none specific. Pure TypeScript, no native modules, so no
build toolchain is required on any platform.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  scroll container (ref, fixed height, overflow-auto)      |
|        |                                                  |
|   useVirtualizer({ count, getScrollElement, estimateSize })|
|        |                                                  |
|        v  virtualizer.getVirtualItems()                   |
|  inner div (height = getTotalSize())                      |
|        +-- row @ translateY(item.start)   visible only    |
|        +-- row @ translateY(item.start)   visible only    |
|                                                           |
|   10,000 items in data --> ~15 nodes in the DOM           |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build a Virtualized List
```tsx
"use client";

import { useRef } from "react";
import { useVirtualizer } from "@tanstack/react-virtual";

export function List({ rows }: { rows: string[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 48,
    overscan: 8,
  });

  return (
    <div ref={parentRef} className="h-96 overflow-auto rounded border">
      <div
        style={{ height: virtualizer.getTotalSize(), position: "relative" }}
      >
        {virtualizer.getVirtualItems().map((item) => (
          <div
            key={item.key}
            className="absolute left-0 top-0 flex w-full items-center px-3"
            style={{
              height: item.size,
              transform: `translateY(${item.start}px)`,
            }}
          >
            {rows[item.index]}
          </div>
        ))}
      </div>
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Dynamic Row Heights
For variable-height content, measure the element instead of estimating.
```tsx
{virtualizer.getVirtualItems().map((item) => (
  <div
    key={item.key}
    ref={virtualizer.measureElement}
    data-index={item.index}
    style={{ transform: `translateY(${item.start}px)` }}
    className="absolute left-0 top-0 w-full"
  >
    {rows[item.index]}
  </div>
))}
```
------------------------------------------------------------------------
## STEP 5 : Infinite Scroll Trigger
```tsx
const items = virtualizer.getVirtualItems();
const last = items[items.length - 1];
if (last && last.index >= rows.length - 1 && hasMore && !loading) {
  loadMore();
}
```
Run this inside an effect keyed on the last visible index so it fires
once per page boundary, not on every scroll frame.
------------------------------------------------------------------------
## STEP 6 : Grid Virtualization
Use two virtualizers — one horizontal, one vertical — and multiply their
virtual items to render only the visible cell window.
```tsx
const colVirt = useVirtualizer({ horizontal: true, count: cols, /* ... */ });
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Give the scroll container a fixed height and overflow-auto
✓ Position rows absolutely with translateY(item.start)
✓ Set the inner wrapper height to getTotalSize()
✓ Use measureElement for dynamic / unknown row heights
✓ Tune overscan (5-10) to trade smoothness for fewer nodes
✓ Keep the virtualized view in a "use client" component
✓ Use item.key for React keys, item.index for data lookup
✓ Debounce infinite-scroll loads to avoid duplicate fetches
```
