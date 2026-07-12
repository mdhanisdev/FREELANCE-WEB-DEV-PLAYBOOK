# JOTAI SETUP [ ATOMIC CLIENT STATE ]
------------------------------------------------------------------------

Jotai models state as small, composable ATOMS. Components subscribe only
to the atoms they read, so updates are surgical. Use it for fine-grained
atomic state and derived values — form fields, toggles, computed UI —
without the boilerplate of a central store.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add jotai
```

------------------------------------------------------------------------

## STEP 2 : Primitive Atoms

An atom is a unit of state. It holds a value and can be read/written.

```ts
// src/atoms/counter.ts
import { atom } from "jotai";

export const countAtom = atom(0);
export const nameAtom = atom("");
```

```tsx
"use client";
import { useAtom } from "jotai";
import { countAtom } from "@/atoms/counter";

export function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

------------------------------------------------------------------------

## STEP 3 : Derived Atoms (Read-only & Writable)

Derived atoms recompute automatically when their dependencies change.

```ts
// src/atoms/cart.ts
import { atom } from "jotai";

export const priceAtom = atom(100);
export const qtyAtom = atom(2);

// read-only derived
export const totalAtom = atom((get) => get(priceAtom) * get(qtyAtom));

// writable derived
export const discountedAtom = atom(
  (get) => get(totalAtom) * 0.9,
  (get, set, newQty: number) => set(qtyAtom, newQty),
);
```

```text
  priceAtom ─┐
             ├─► totalAtom ─► discountedAtom
  qtyAtom  ──┘   (auto-recomputes on dependency change)
```

------------------------------------------------------------------------

## STEP 4 : Read-only / Write-only Hooks

```tsx
"use client";
import { useAtomValue, useSetAtom } from "jotai";
import { totalAtom, qtyAtom } from "@/atoms/cart";

export function Total() {
  const total = useAtomValue(totalAtom); // read only, no setter
  const setQty = useSetAtom(qtyAtom); // write only, no re-render on read
  return (
    <div>
      <span>Total: {total}</span>
      <button onClick={() => setQty((q) => q + 1)}>Add</button>
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Provider (App Router)

For most apps the default global store works without a Provider. Add a
`Provider` when you need isolated store scopes (e.g. per-route, tests,
or SSR request isolation).

```tsx
// src/atoms/provider.tsx
"use client";
import { Provider } from "jotai";

export function AtomProvider({ children }: { children: React.ReactNode }) {
  return <Provider>{children}</Provider>;
}
```

```tsx
// src/app/layout.tsx
import { AtomProvider } from "@/atoms/provider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <AtomProvider>{children}</AtomProvider>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Persisted Atom (localStorage)

```ts
// src/atoms/theme.ts
import { atomWithStorage } from "jotai/utils";

export const themeAtom = atomWithStorage<"light" | "dark">("theme", "light");
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep atoms small and single-purpose; compose via derived atoms
✓ Use useAtomValue for reads, useSetAtom for writes
✓ Let derived atoms compute values — never duplicate state
✓ Reach for atomWithStorage instead of manual localStorage
✓ Add a Provider only when you need scoped/isolated stores
✓ Mark atom-consuming components "use client"
✓ Prefer Jotai for fine-grained atomic UI state
✓ Use server-data tools (TanStack Query/SWR) for remote data
```
