# REDUX TOOLKIT SETUP [ PREDICTABLE CLIENT STATE ]
------------------------------------------------------------------------

Redux Toolkit (RTK) is the official, opinionated way to write Redux. Use
it for LARGE apps with complex, interrelated client state, strict
action traceability, or when a team already knows Redux. For simple
global UI state prefer Zustand; for server data prefer RTK Query /
TanStack Query.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @reduxjs/toolkit react-redux
```

------------------------------------------------------------------------

## STEP 2 : Create A Slice

```ts
// src/store/counter-slice.ts
import { createSlice, type PayloadAction } from "@reduxjs/toolkit";

interface CounterState {
  value: number;
}

const initialState: CounterState = { value: 0 };

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1; // Immer: safe "mutation"
    },
    addBy: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});

export const { increment, addBy } = counterSlice.actions;
export default counterSlice.reducer;
```

------------------------------------------------------------------------

## STEP 3 : configureStore

```ts
// src/store/index.ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./counter-slice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

------------------------------------------------------------------------

## STEP 4 : Typed Hooks

```ts
// src/store/hooks.ts
import { useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./index";

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

------------------------------------------------------------------------

## STEP 5 : Provider (App Router)

The store is client-only. Wrap children in a "use client" provider.

```tsx
// src/store/provider.tsx
"use client";
import { Provider } from "react-redux";
import { store } from "./index";

export function StoreProvider({ children }: { children: React.ReactNode }) {
  return <Provider store={store}>{children}</Provider>;
}
```

```tsx
// src/app/layout.tsx
import { StoreProvider } from "@/store/provider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <StoreProvider>{children}</StoreProvider>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Consume In A Component

```tsx
"use client";
import { useAppDispatch, useAppSelector } from "@/store/hooks";
import { increment, addBy } from "@/store/counter-slice";

export function Counter() {
  const value = useAppSelector((s) => s.counter.value);
  const dispatch = useAppDispatch();
  return (
    <div>
      <span>{value}</span>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(addBy(5))}>+5</button>
    </div>
  );
}
```

------------------------------------------------------------------------

## RTK QUERY (SERVER DATA)

RTK Query ships inside `@reduxjs/toolkit` and handles fetching, caching,
and invalidation — an alternative to TanStack Query if you already run
Redux.

```text
  createApi ──► endpoints ──► auto-generated hooks
                                │
              useGetPostsQuery ─┘   useAddPostMutation
              (cache + refetch)     (invalidatesTags → refetch)
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always use configureStore, never legacy createStore
✓ Write logic in createSlice; let Immer handle immutability
✓ Export typed useAppDispatch / useAppSelector, not raw hooks
✓ Keep the Provider in a "use client" boundary
✓ Select minimal state; avoid returning new objects each render
✓ Use RTK Query for server data instead of hand-rolled thunks
✓ One slice per domain; compose in the root reducer
✓ Reach for RTK on large teams/apps; use Zustand for simple UI state
```
