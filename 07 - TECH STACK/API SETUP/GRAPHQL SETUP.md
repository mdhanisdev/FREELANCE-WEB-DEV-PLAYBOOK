# GRAPHQL SETUP [ CODE-FIRST SCHEMA API WITH YOGA + POTHOS ]
------------------------------------------------------------------------

GraphQL exposes ONE endpoint where clients request exactly the fields
they need. We use GraphQL Yoga as the server (a Next.js route handler)
and Pothos as a code-first, fully typed schema builder.

```text
  many clients            single endpoint         resolvers
 ┌──────────┐   query {   ┌───────────────┐      ┌──────────┐
 │ web / ios│──────────▶  │  /api/graphql │────▶ │ Pothos   │
 │ 3rd-party│   user{...} │  (Yoga)       │      │ schema   │
 └──────────┘◀──────────  └───────────────┘◀──── └──────────┘
   pick fields   typed json     one round trip
```

------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add graphql graphql-yoga @pothos/core zod
```

------------------------------------------------------------------------
## STEP 2 : Build The Schema (Pothos, code-first)

Pothos infers TypeScript types from your builder calls — no `.graphql`
files, no codegen for the server side.

```ts
// server/graphql/builder.ts
import SchemaBuilder from "@pothos/core";

export const builder = new SchemaBuilder<{
  Context: { userId: string | null };
}>({});
```

```ts
// server/graphql/schema.ts
import { z } from "zod";
import { builder } from "./builder";

// Object type
const User = builder.objectRef<{ id: string; email: string }>("User");
builder.objectType(User, {
  fields: (t) => ({
    id: t.exposeString("id"),
    email: t.exposeString("email"),
  }),
});

const CreateUser = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});

// Query
builder.queryType({
  fields: (t) => ({
    user: t.field({
      type: User,
      args: { id: t.arg.string({ required: true }) },
      resolve: (_p, args) => db.user.findUnique({ where: { id: args.id } }),
    }),
  }),
});

// Mutation (with Zod validation + authz)
builder.mutationType({
  fields: (t) => ({
    createUser: t.field({
      type: User,
      args: {
        email: t.arg.string({ required: true }),
        name: t.arg.string({ required: true }),
      },
      resolve: (_p, args, ctx) => {
        if (!ctx.userId) throw new Error("Unauthorized");
        const parsed = CreateUser.safeParse(args);
        if (!parsed.success) throw new Error("Invalid input");
        return db.user.create({ data: parsed.data });
      },
    }),
  }),
});

export const schema = builder.toSchema();
```

------------------------------------------------------------------------
## STEP 3 : The Yoga Route Handler

Yoga plugs straight into the App Router fetch API.

```ts
// app/api/graphql/route.ts
import { createYoga } from "graphql-yoga";
import { schema } from "@/server/graphql/schema";
import { auth } from "@/lib/auth";

const { handleRequest } = createYoga<{
  request: Request;
}>({
  schema,
  graphqlEndpoint: "/api/graphql",
  fetchAPI: { Response },
  context: async () => {
    const session = await auth();
    return { userId: session?.user.id ?? null };
  },
});

export { handleRequest as GET, handleRequest as POST };
```

Visit `/api/graphql` in the browser for the built-in GraphiQL explorer.

------------------------------------------------------------------------
## STEP 4 : Client Usage

For simple apps a typed `fetch` wrapper is enough — no client library.

```ts
// lib/gql.ts
export async function gql<T>(query: string, variables?: unknown): Promise<T> {
  const res = await fetch("/api/graphql", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ query, variables }),
  });
  const json = await res.json();
  if (json.errors) throw new Error(json.errors[0].message);
  return json.data as T;
}
```

```tsx
"use client";
import { useEffect, useState } from "react";
import { gql } from "@/lib/gql";

const QUERY = `query ($id: String!) { user(id: $id) { email } }`;

export function User({ id }: { id: string }) {
  const [email, setEmail] = useState("user@example.com");
  useEffect(() => {
    gql<{ user: { email: string } }>(QUERY, { id }).then((d) =>
      setEmail(d.user.email),
    );
  }, [id]);
  return <span>{email}</span>;
}
```

For larger apps, add Apollo Client or urql plus GraphQL Code Generator
to get typed client hooks.

------------------------------------------------------------------------
## WHEN TO CHOOSE GraphQL

```text
✓ Pick GraphQL for MANY diverse clients (web, mobile, 3rd-party)
✓ Pick it when clients need to shape/select fields to cut over-fetching
✓ Pick it when you want a language-agnostic, self-documenting schema
✗ Skip it for a TS-only monorepo — tRPC is lighter with less ceremony
✗ Skip it for trivial CRUD — plain route handlers are simpler
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use Pothos code-first — types flow from the builder, no server codegen
✓ Build context per-request and put auth (userId/session) there
✓ Validate mutation args with Zod safeParse before touching the db
✓ Authorize inside resolvers using ctx — throw on missing userId
✓ Expose one /api/graphql endpoint via a Yoga route handler
✓ Disable GraphiQL in production or gate it behind auth
✓ Keep resolvers thin — delegate to a service/data layer
✓ Add query depth/complexity limits to prevent abusive queries
```
