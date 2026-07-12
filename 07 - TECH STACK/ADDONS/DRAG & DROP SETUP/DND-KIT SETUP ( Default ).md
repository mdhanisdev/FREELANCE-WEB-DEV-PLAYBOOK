# DND-KIT SETUP [ DRAG & DROP ]
------------------------------------------------------------------------
dnd-kit is a modern, lightweight, accessible drag-and-drop toolkit for
React. It is sensor-based (pointer, keyboard, touch), tree-shakeable,
and unopinionated about styling — the default pick for sortable lists,
kanban boards, and reorderable grids in a Tailwind + TS stack.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```
Windows note: none specific. dnd-kit is pure TypeScript with no native
build step, so no C++ toolchain is required.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <DndContext sensors onDragEnd>                           |
|        |                                                  |
|        +-- <SortableContext items=[ids]>                  |
|        |        |                                         |
|        |        +-- useSortable(id) --> attributes,       |
|        |                 listeners, setNodeRef, transform |
|        |                                                  |
|        +-- Sensors: Pointer + Keyboard (a11y)             |
|                                                           |
|  onDragEnd(active,over) --> arrayMove(items) --> setState |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Set Up the Context and Sensors
```tsx
"use client";

import { useState } from "react";
import {
  DndContext, closestCenter, PointerSensor, KeyboardSensor,
  useSensor, useSensors, type DragEndEvent,
} from "@dnd-kit/core";
import {
  SortableContext, arrayMove, verticalListSortingStrategy,
  sortableKeyboardCoordinates,
} from "@dnd-kit/sortable";

export function SortableList() {
  const [items, setItems] = useState(["a", "b", "c", "d"]);
  const sensors = useSensors(
    useSensor(PointerSensor, { activationConstraint: { distance: 6 } }),
    useSensor(KeyboardSensor, { coordinateGetter: sortableKeyboardCoordinates })
  );

  function onDragEnd({ active, over }: DragEndEvent) {
    if (over && active.id !== over.id) {
      setItems((prev) =>
        arrayMove(prev, prev.indexOf(active.id as string),
          prev.indexOf(over.id as string))
      );
    }
  }

  return (
    <DndContext sensors={sensors} collisionDetection={closestCenter}
      onDragEnd={onDragEnd}>
      <SortableContext items={items} strategy={verticalListSortingStrategy}>
        <ul className="space-y-2">
          {items.map((id) => <Item key={id} id={id} />)}
        </ul>
      </SortableContext>
    </DndContext>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Build a Sortable Item
```tsx
"use client";

import { useSortable } from "@dnd-kit/sortable";
import { CSS } from "@dnd-kit/utilities";

function Item({ id }: { id: string }) {
  const { attributes, listeners, setNodeRef, transform, transition,
    isDragging } = useSortable({ id });

  return (
    <li
      ref={setNodeRef}
      style={{ transform: CSS.Transform.toString(transform), transition }}
      {...attributes}
      {...listeners}
      className={`cursor-grab rounded border bg-white p-3 ${
        isDragging ? "opacity-50" : ""}`}
    >
      {id}
    </li>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Add a Drag Overlay (No Layout Jump)
```tsx
import { DragOverlay } from "@dnd-kit/core";

<DragOverlay>
  {activeId ? <div className="rounded border bg-white p-3">{activeId}</div>
    : null}
</DragOverlay>
```
Track `activeId` via `onDragStart` and clear it in `onDragEnd`.
------------------------------------------------------------------------
## STEP 6 : Persist the New Order
Send the reordered id array to a Server Action; store positions as an
integer column so re-inserts do not require rewriting every row.
```ts
"use server";
export async function saveOrder(ids: string[]) {
  // await db.$transaction(ids.map((id, i) => update(id, { position: i })));
}
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Always register the KeyboardSensor for accessible reordering
✓ Set an activationConstraint so clicks are not swallowed as drags
✓ Use CSS.Transform.toString for GPU-friendly movement
✓ Give SortableContext stable string/number ids, never array index
✓ Use DragOverlay to avoid layout shift while dragging
✓ Keep drag logic in a "use client" component
✓ Persist order via a Server Action, ideally in one transaction
✓ Memoize item components to prevent whole-list re-renders
```
