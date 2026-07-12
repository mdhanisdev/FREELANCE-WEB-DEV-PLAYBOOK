# REACT-WINDOW SETUP [ VIRTUALIZATION ]
------------------------------------------------------------------------
react-window is a tiny, focused virtualization library that renders
fixed- or variable-size windowed lists and grids. It is more prescriptive
than TanStack Virtual (it owns the markup) but its small API makes it
fast to adopt for straightforward, uniformly-sized lists.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add react-window
pnpm add -D @types/react-window
```
Windows note: none specific. Types ship separately from DefinitelyTyped;
install them as a dev dependency for a fully typed API in TypeScript.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <FixedSizeList height width itemCount itemSize>          |
|        |                                                  |
|        +-- passes {index, style} to your Row component    |
|        +-- style has absolute position + top offset       |
|                                                           |
|  Only rows in the viewport (+ overscanCount) are mounted  |
|                                                           |
|  100,000 rows --> a handful of mounted DOM nodes          |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build a Fixed-Size List
The `style` prop is required — it positions each row absolutely.
```tsx
"use client";

import { FixedSizeList, type ListChildComponentProps } from "react-window";

function Row({ index, style }: ListChildComponentProps<string[]>) {
  return (
    <div style={style} className="flex items-center border-b px-3">
      Row {index}
    </div>
  );
}

export function List({ rows }: { rows: string[] }) {
  return (
    <FixedSizeList
      height={384}
      width="100%"
      itemCount={rows.length}
      itemSize={48}
      itemData={rows}
      overscanCount={8}
      className="rounded border"
    >
      {Row}
    </FixedSizeList>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Variable Row Heights
```tsx
import { VariableSizeList } from "react-window";

const getSize = (index: number) => (index % 2 ? 72 : 48);

<VariableSizeList height={384} width="100%" itemCount={rows.length}
  itemSize={getSize}>
  {Row}
</VariableSizeList>;
```
Call `listRef.current?.resetAfterIndex(i)` when a measured size changes.
------------------------------------------------------------------------
## STEP 5 : Responsive Sizing with AutoSizer
react-window needs explicit pixel dimensions; pair it with AutoSizer.
```bash
pnpm add react-virtualized-auto-sizer
```
```tsx
import AutoSizer from "react-virtualized-auto-sizer";

<div className="h-96">
  <AutoSizer>
    {({ height, width }) => (
      <FixedSizeList height={height} width={width} itemCount={rows.length}
        itemSize={48}>
        {Row}
      </FixedSizeList>
    )}
  </AutoSizer>
</div>;
```
------------------------------------------------------------------------
## STEP 6 : Infinite Loading
Track the last rendered index via `onItemsRendered` and fetch the next
page when it approaches `itemCount`, guarding against duplicate loads.
```tsx
<FixedSizeList
  onItemsRendered={({ visibleStopIndex }) => {
    if (visibleStopIndex >= rows.length - 1 && !loading) loadMore();
  }}
/>;
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Always spread the injected style prop onto each Row's root
✓ Provide explicit height/width — use AutoSizer to make it fluid
✓ Pass itemData instead of closing over parent scope arrays
✓ Call resetAfterIndex after variable sizes change
✓ Keep the list in a "use client" component
✓ Tune overscanCount to balance smoothness and node count
✓ Guard onItemsRendered loads against duplicate fetches
✓ Prefer TanStack Virtual when you need headless / dynamic markup
```
