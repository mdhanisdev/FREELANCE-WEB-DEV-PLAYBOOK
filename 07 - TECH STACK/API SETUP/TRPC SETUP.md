# TRPC SETUP [ END-TO-END TYPESAFE API LAYER ]
------------------------------------------------------------------------

tRPC gives you a fully typed client↔server bridge with NO code
generation and NO schema files. You write server procedures, and the
client infers their input/output types automatically.

```text
   CLIENT                          SERVER
 ┌────────┐   types flow ↑↑↑     ┌──────────┐
 │ trpc.  │◀────────────────────▶│ router   │
 │ user   │   one fetch call     │ + Zod    │
 │ .get() │────────────────────▶ │ procedure│
 └────────┘   no codegen          └──────────┘
```

Choose tRPC when the SAME team owns both ends and both are TypeScript.

------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add @trpc/server @trpc/client @trpc/react-query @tanstack/react-query zod
```

------------------------------------------------------------------------
## STEP 2 : Init Router + Context

Context is built per-request and holds things like the session/db.

```ts
// server/trpc/context.ts
import { auth } from "@/lib/auth";

export async function createContext() {
  const session = await auth();
  return { session };
}
export type Context = Awaited<ReturnType<typeof createContext>>;
```

```ts
// server/trpc/trpc.ts
import { initTRPC, TRPCError } from "@trpc/server";
import type { Context } from "./context";

const t = initTRPC.context<Context>().create();

export const router = t.router;
export const publicProcedure = t.procedure;

// Reusable auth middleware.
export const protectedProcedure = t.procedure.use(({ ctx, next }) => {
  if (!ctx.session) throw new TRPCError({ code: "UNAUTHORIZED" });
  return next({ ctx: { session: ctx.session } });
});
```

------------------------------------------------------------------------
## STEP 3 : Procedures With Zod

`.input()` takes a Zod schema; the parsed value arrives typed in
`input`. Invalid payloads reject before your resolver runs.

```ts
// server/trpc/routers/user.ts
import { z } from "zod";
import { router, publicProcedure, protectedProcedure } from "../trpc";

export const userRouter = router({
  byId: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(({ input }) => db.user.findUnique({ where: { id: input.id } })),

  create: protectedProcedure
    .input(z.object({ email: z.string().email(), name: z.string().min(1) }))
    .mutation(({ input }) => db.user.create({ data: input })),
});
```

```ts
// server/trpc/routers/_app.ts
import { router } from "../trpc";
import { userRouter } from "./user";

export const appRouter = router({ user: userRouter });
export type AppRouter = typeof appRouter; // <- exported for the client
```

------------------------------------------------------------------------
## STEP 4 : App Router Adapter

One catch-all route handler serves every procedure.

```ts
// app/api/trpc/[trpc]/route.ts
import { fetchRequestHandler } from "@trpc/server/adapters/fetch";
import { appRouter } from "@/server/trpc/routers/_app";
import { createContext } from "@/server/trpc/context";

const handler = (req: Request) =>
  fetchRequestHandler({
    endpoint: "/api/trpc",
    req,
    router: appRouter,
    createContext,
  });

export { handler as GET, handler as POST };
```

------------------------------------------------------------------------
## STEP 5 : Client Setup

```ts
// lib/trpc.ts
import { createTRPCReact } from "@trpc/react-query";
import type { AppRouter } from "@/server/trpc/routers/_app";

export const trpc = createTRPCReact<AppRouter>();
```

```tsx
// app/providers.tsx
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { httpBatchLink } from "@trpc/client";
import { useState } from "react";
import { trpc } from "@/lib/trpc";

export function Providers({ children }: { children: React.ReactNode }) {
  const [qc] = useState(() => new QueryClient());
  const [client] = useState(() =>
    trpc.createClient({ links: [httpBatchLink({ url: "/api/trpc" })] }),
  );
  return (
    <trpc.Provider client={client} queryClient={qc}>
      <QueryClientProvider client={qc}>{children}</QueryClientProvider>
    </trpc.Provider>
  );
}
```

Consume it — fully typed, autocompleted, no `any`:

```tsx
"use client";
import { trpc } from "@/lib/trpc";

export function User({ id }: { id: string }) {
  const { data, isLoading } = trpc.user.byId.useQuery({ id });
  if (isLoading) return <p>…</p>;
  return <span>{data?.email ?? "user@example.com"}</span>;
}
```

------------------------------------------------------------------------
## WHEN TO CHOOSE tRPC

```text
✓ Pick tRPC when TypeScript owns BOTH client and server (monorepo)
✓ Pick it when you want zero codegen and instant type inference
✗ Avoid it for public APIs consumed by non-TS / third-party clients
✗ Avoid it when you need a language-agnostic schema (use GraphQL/REST)
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Validate every procedure with a Zod .input() schema
✓ Put auth in a protectedProcedure middleware, not each resolver
✓ Build context per-request (session + db) and type it
✓ Export AppRouter type — the client imports the TYPE only, never code
✓ Use httpBatchLink to collapse many calls into one request
✓ Throw TRPCError with proper codes (UNAUTHORIZED, NOT_FOUND)
✓ Keep routers small and merge them in _app.ts
✓ Never import server routers into client code — only the type
```
