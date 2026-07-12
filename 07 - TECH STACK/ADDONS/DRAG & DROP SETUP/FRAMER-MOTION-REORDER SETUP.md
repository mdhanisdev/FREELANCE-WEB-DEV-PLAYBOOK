# FRAMER-MOTION-REORDER SETUP [ DRAG & DROP ]
------------------------------------------------------------------------
Framer Motion (now the `motion` package) ships a Reorder primitive that
gives you animated, draggable lists with almost no code. Choose it when
your list is one-dimensional and you want buttery layout animations for
free. For kanban / grid / nested trees, prefer dnd-kit instead.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add motion
```
Windows note: none specific. The `motion` package is pure JS; imports
use `motion/react` — keep casing exact so case-sensitive CI matches.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <Reorder.Group axis="y" values={items} onReorder>        |
|        |                                                  |
|        +-- <Reorder.Item value={item}>   draggable row    |
|        +-- <Reorder.Item value={item}>   draggable row    |
|                                                           |
|   drag gesture --> onReorder(newOrder) --> setState       |
|   layout animation handled automatically by motion        |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Reorderable List
```tsx
"use client";

import { useState } from "react";
import { Reorder } from "motion/react";

export function List() {
  const [items, setItems] = useState(["Alpha", "Bravo", "Charlie"]);

  return (
    <Reorder.Group
      axis="y"
      values={items}
      onReorder={setItems}
      className="space-y-2"
    >
      {items.map((item) => (
        <Reorder.Item
          key={item}
          value={item}
          className="cursor-grab rounded border bg-white p-3 active:cursor-grabbing"
        >
          {item}
        </Reorder.Item>
      ))}
    </Reorder.Group>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Add a Dedicated Drag Handle
Use `useDragControls` so only the handle initiates a drag.
```tsx
"use client";

import { Reorder, useDragControls } from "motion/react";

function Row({ item }: { item: string }) {
  const controls = useDragControls();
  return (
    <Reorder.Item value={item} dragListener={false} dragControls={controls}
      className="flex items-center gap-2 rounded border bg-white p-3">
      <span
        onPointerDown={(e) => controls.start(e)}
        className="cursor-grab select-none"
      >
        ⠿
      </span>
      {item}
    </Reorder.Item>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Animate Enter and Exit
Wrap items in `AnimatePresence` for add/remove transitions.
```tsx
import { AnimatePresence } from "motion/react";

<AnimatePresence initial={false}>
  {items.map((i) => (
    <Reorder.Item key={i} value={i}
      initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} />
  ))}
</AnimatePresence>
```
------------------------------------------------------------------------
## STEP 6 : Persist the Order
`onReorder` fires with the full new array on every move. Debounce before
writing to a Server Action so a fast drag does not spam the network.
```ts
"use server";
export async function saveOrder(items: string[]) {
  // await db.reorder(items);
}
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Use stable, unique key AND value on every Reorder.Item
✓ Set dragListener={false} + dragControls for handle-only dragging
✓ Keep the list in a "use client" component
✓ Prefer this only for 1D lists — use dnd-kit for boards / grids
✓ Debounce onReorder before persisting to avoid network spam
✓ Wrap in AnimatePresence for smooth add / remove animations
✓ Match axis prop ("y"/"x") to your visual layout direction
✓ Respect prefers-reduced-motion for accessibility
```
