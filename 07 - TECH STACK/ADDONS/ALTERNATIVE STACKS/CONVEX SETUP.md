# CONVEX SETUP [ ALTERNATIVE STACKS ]

------------------------------------------------------------------------

Convex is a reactive backend: you write TypeScript query and mutation
functions, and the client subscribes to them so the UI updates
automatically when data changes. It replaces the database, API layer, and
websocket plumbing with one type-safe system.

------------------------------------------------------------------------

## STEP 1 : Install And Initialize

```bash
pnpm add convex
pnpm dlx convex dev
```

The first `convex dev` run logs you in, provisions a dev deployment, and
creates the `convex/` folder. Leave it running — it hot-pushes function
changes and codegen.

------------------------------------------------------------------------

## STEP 2 : Define A Schema

```ts
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  tasks: defineTable({
    text: v.string(),
    done: v.boolean(),
    userId: v.string(),
  }).index("by_user", ["userId"]),
});
```

------------------------------------------------------------------------

## STEP 3 : Write Queries And Mutations

```ts
// convex/tasks.ts
import { query, mutation } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: { userId: v.string() },
  handler: async (ctx, { userId }) =>
    ctx.db.query("tasks").withIndex("by_user", (q) =>
      q.eq("userId", userId)).collect(),
});

export const add = mutation({
  args: { text: v.string(), userId: v.string() },
  handler: async (ctx, { text, userId }) =>
    ctx.db.insert("tasks", { text, done: false, userId }),
});
```

------------------------------------------------------------------------

## STEP 4 : Wire The Provider

```tsx
// app/providers.tsx
"use client";
import { ConvexProvider, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function Providers({ children }: { children: React.ReactNode }) {
  return <ConvexProvider client={convex}>{children}</ConvexProvider>;
}
```

------------------------------------------------------------------------

## STEP 5 : Consume Reactively

```tsx
"use client";
import { useQuery, useMutation } from "convex/react";
import { api } from "@/convex/_generated/api";

export function Tasks({ userId }: { userId: string }) {
  const tasks = useQuery(api.tasks.list, { userId }); // live, auto-updates
  const add = useMutation(api.tasks.add);
  // no refetch, no cache invalidation — Convex pushes changes
  return <button onClick={() => add({ text: "New", userId })}>Add</button>;
}
```

```text
   Client useQuery ──subscribe──▶ Convex function ──▶ Convex DB
        ▲                                                 │
        └────────── push on any matching write ───────────┘

   A mutation elsewhere re-runs the query and re-renders the UI.
```

------------------------------------------------------------------------

## STEP 6 : Deploy

```bash
pnpm dlx convex deploy        # push functions to production
# set NEXT_PUBLIC_CONVEX_URL to the prod deployment in Vercel
```

Windows note: keep `convex dev` running in its own terminal; it watches
files and closing it stops the live codegen that types `api`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Model data in schema.ts and add indexes for every query filter
✓ Query by index, never scan full tables in handlers
✓ Let useQuery drive the UI — avoid manual refetch logic
✓ Keep convex dev running for continuous type generation
✓ Do authorization inside handlers using ctx identity
✓ Use mutations for writes, actions for external API calls
✓ Commit the generated _generated folder is optional — regen in CI
✓ Separate dev and prod deployments via env URL
✓ Prefer Convex when live, reactive data is core to the app
```
