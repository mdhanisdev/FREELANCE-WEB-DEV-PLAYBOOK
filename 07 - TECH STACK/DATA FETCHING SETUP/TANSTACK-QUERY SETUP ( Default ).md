# TANSTACK QUERY SETUP [ SERVER STATE / DATA FETCHING ]
------------------------------------------------------------------------

TanStack Query manages SERVER STATE: fetching, caching, background
refetch, and invalidation. Use it for client-side data that changes
(dashboards, infinite lists, mutations). For static or first-paint data
prefer Next.js Server Components; use Query for interactive,
client-driven data.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @tanstack/react-query
pnpm add -D @tanstack/react-query-devtools
```

------------------------------------------------------------------------

## STEP 2 : QueryClient Provider (App Router)

Create the client inside state so it is stable per browser session and
never shared across server requests.

```tsx
// src/app/providers.tsx
"use client";
import { useState } from "react";
import {
  QueryClient,
  QueryClientProvider,
} from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

export function Providers({ children }: { children: React.ReactNode }) {
  const [client] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: { staleTime: 60_000, refetchOnWindowFocus: false },
        },
      }),
  );
  return (
    <QueryClientProvider client={client}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```tsx
// src/app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : useQuery (Reading Data)

```tsx
"use client";
import { useQuery } from "@tanstack/react-query";

interface User {
  id: string;
  email: string;
}

async function fetchUsers(): Promise<User[]> {
  const res = await fetch("/api/users");
  if (!res.ok) throw new Error("Failed to load users");
  return res.json();
}

export function UserList() {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  if (isPending) return <p>Loading…</p>;
  if (isError) return <p>{error.message}</p>;
  return (
    <ul>
      {data.map((u) => (
        <li key={u.id}>{u.email}</li>
      ))}
    </ul>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : useMutation + Invalidation

After a write, invalidate the affected query key so cached data refetches.

```text
  useMutation.mutate() ──► POST /api/users
        │ onSuccess
        ▼
  queryClient.invalidateQueries({ queryKey: ["users"] })
        │
        ▼
  ["users"] marked stale ──► automatic background refetch ──► UI updates
```

```tsx
"use client";
import { useMutation, useQueryClient } from "@tanstack/react-query";

export function AddUser() {
  const qc = useQueryClient();
  const { mutate, isPending } = useMutation({
    mutationFn: (email: string) =>
      fetch("/api/users", {
        method: "POST",
        body: JSON.stringify({ email }),
      }).then((r) => r.json()),
    onSuccess: () => {
      qc.invalidateQueries({ queryKey: ["users"] });
    },
  });

  return (
    <button
      disabled={isPending}
      onClick={() => mutate("user@example.com")}
    >
      Add user
    </button>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : staleTime vs gcTime

```text
✓ staleTime  — how long data is "fresh"; no refetch while fresh
✓ gcTime     — how long unused cache is kept before garbage collection
```

Raise `staleTime` for data that rarely changes to cut network chatter.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Create QueryClient in useState, never at module scope
✓ Use structured, unique queryKeys (["users", id])
✓ Throw in queryFn on non-ok responses so isError works
✓ Invalidate related keys in mutation onSuccess
✓ Tune staleTime per resource; disable focus refetch if noisy
✓ Prefer Server Components for first-paint/static data
✓ Use TanStack Query for interactive, client-mutated data
✓ Keep server state OUT of Zustand/Redux
```
