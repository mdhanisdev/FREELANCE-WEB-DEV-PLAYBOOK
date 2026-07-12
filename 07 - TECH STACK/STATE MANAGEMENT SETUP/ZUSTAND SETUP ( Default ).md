# ZUSTAND SETUP [ CLIENT STATE MANAGEMENT ]
------------------------------------------------------------------------

Zustand is a small, fast, unopinionated state manager. Use it for
GLOBAL UI STATE (theme, sidebar, modals, wizards) — NOT for server
data (use TanStack Query / SWR / Server Components for that).

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add zustand
```

------------------------------------------------------------------------

## STEP 2 : Create A Basic Store

```ts
// src/stores/ui-store.ts
import { create } from "zustand";

interface UIState {
  sidebarOpen: boolean;
  toggleSidebar: () => void;
  openSidebar: () => void;
  closeSidebar: () => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  openSidebar: () => set({ sidebarOpen: true }),
  closeSidebar: () => set({ sidebarOpen: false }),
}));
```

------------------------------------------------------------------------

## STEP 2 : Consume With Selectors (Avoid Re-renders)

Always select the SLICE you need. Selecting the whole store re-renders
the component on every state change.

```tsx
"use client";
import { useUIStore } from "@/stores/ui-store";

export function SidebarToggle() {
  // ✓ only re-renders when sidebarOpen changes
  const open = useUIStore((s) => s.sidebarOpen);
  const toggle = useUIStore((s) => s.toggleSidebar);
  return <button onClick={toggle}>{open ? "Close" : "Open"}</button>;
}
```

------------------------------------------------------------------------

## STEP 3 : Persist Middleware (localStorage)

```ts
// src/stores/theme-store.ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";

type Theme = "light" | "dark" | "system";

interface ThemeState {
  theme: Theme;
  setTheme: (t: Theme) => void;
}

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: "system",
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: "theme-storage",
      storage: createJSONStorage(() => localStorage),
      partialize: (s) => ({ theme: s.theme }),
    },
  ),
);
```

------------------------------------------------------------------------

## STEP 4 : Slices Pattern (Split A Large Store)

```ts
// src/stores/slices.ts
import { create } from "zustand";
import type { StateCreator } from "zustand";

interface CartSlice {
  items: string[];
  addItem: (id: string) => void;
}
interface FilterSlice {
  query: string;
  setQuery: (q: string) => void;
}

const createCart: StateCreator<CartSlice & FilterSlice, [], [], CartSlice> =
  (set) => ({
    items: [],
    addItem: (id) => set((s) => ({ items: [...s.items, id] })),
  });

const createFilter: StateCreator<CartSlice & FilterSlice, [], [], FilterSlice> =
  (set) => ({
    query: "",
    setQuery: (query) => set({ query }),
  });

export const useAppStore = create<CartSlice & FilterSlice>()((...a) => ({
  ...createCart(...a),
  ...createFilter(...a),
}));
```

------------------------------------------------------------------------

## STEP 5 : SSR Safety (Next.js App Router)

Stores are module singletons — do NOT share them across requests on the
server. Read persisted state only after mount to avoid hydration
mismatch.

```text
  Server render          Client hydrate
  ─────────────          ──────────────
  default state   ──►    mount effect  ──►  apply persisted value
  (no localStorage)      (localStorage available)
```

```tsx
"use client";
import { useEffect, useState } from "react";
import { useThemeStore } from "@/stores/theme-store";

export function useHydratedTheme() {
  const [mounted, setMounted] = useState(false);
  const theme = useThemeStore((s) => s.theme);
  useEffect(() => setMounted(true), []);
  return mounted ? theme : "system";
}
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use for global CLIENT/UI state, never for server data
✓ Always read via narrow selectors: useStore((s) => s.field)
✓ Keep actions inside the store, colocated with state
✓ Use persist middleware only for values worth restoring
✓ partialize to persist a subset, never the whole store
✓ Split big stores with the slices pattern
✓ Guard localStorage reads behind mount to prevent hydration errors
✓ Type the store with an explicit interface
✓ Mark consuming components "use client"
```
