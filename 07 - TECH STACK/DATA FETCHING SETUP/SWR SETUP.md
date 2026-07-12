# SWR SETUP [ SERVER STATE / DATA FETCHING ]
------------------------------------------------------------------------

SWR (stale-while-revalidate) is Vercel's lightweight data-fetching hook.
It returns cached data immediately, then revalidates in the background.
Use it for simple client-side reads with minimal config. For heavy
mutation workflows TanStack Query offers more; for first-paint data
prefer Next.js Server Components.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add swr
```

------------------------------------------------------------------------

## STEP 2 : Define A Fetcher

```ts
// src/lib/fetcher.ts
export const fetcher = async <T>(url: string): Promise<T> => {
  const res = await fetch(url);
  if (!res.ok) throw new Error("Request failed");
  return res.json();
};
```

------------------------------------------------------------------------

## STEP 3 : useSWR (Reading Data)

```tsx
"use client";
import useSWR from "swr";
import { fetcher } from "@/lib/fetcher";

interface User {
  id: string;
  email: string;
}

export function UserList() {
  const { data, error, isLoading } = useSWR<User[]>("/api/users", fetcher);

  if (isLoading) return <p>Loading…</p>;
  if (error) return <p>Failed to load</p>;
  return (
    <ul>
      {data?.map((u) => (
        <li key={u.id}>{u.email}</li>
      ))}
    </ul>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Revalidation Flow

```text
  useSWR("/api/users")
        │
        ├─► return cached data instantly (stale)
        │
        └─► refetch in background ──► update cache ──► re-render (fresh)
              triggers: focus • reconnect • interval • mutate()
```

------------------------------------------------------------------------

## STEP 5 : Mutations (Optimistic Update)

`mutate` updates the cache and revalidates. Pass data + a rollback-safe
option for optimistic UI.

```tsx
"use client";
import useSWR from "swr";
import { fetcher } from "@/lib/fetcher";

export function AddUser() {
  const { data, mutate } = useSWR("/api/users", fetcher);

  async function add() {
    const email = "user@example.com";
    await mutate(
      async () => {
        await fetch("/api/users", {
          method: "POST",
          body: JSON.stringify({ email }),
        });
        return fetcher("/api/users");
      },
      {
        optimisticData: [...(data ?? []), { id: "temp", email }],
        rollbackOnError: true,
        revalidate: true,
      },
    );
  }

  return <button onClick={add}>Add user</button>;
}
```

------------------------------------------------------------------------

## STEP 6 : Global Config

```tsx
// src/app/providers.tsx
"use client";
import { SWRConfig } from "swr";
import { fetcher } from "@/lib/fetcher";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <SWRConfig
      value={{
        fetcher,
        revalidateOnFocus: false,
        dedupingInterval: 5_000,
        errorRetryCount: 2,
      }}
    >
      {children}
    </SWRConfig>
  );
}
```

With a global fetcher set, hooks simplify to `useSWR("/api/users")`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use a single typed fetcher; set it once via SWRConfig
✓ Throw in the fetcher on non-ok responses so error is populated
✓ Key by the request URL; use arrays for parameterized keys
✓ Use mutate with optimisticData + rollbackOnError for snappy UI
✓ Tune dedupingInterval and revalidateOnFocus to cut requests
✓ Prefer Server Components for first-paint/static data
✓ Reach for SWR when you want minimal, read-heavy fetching
✓ Keep server data out of client state stores
```
