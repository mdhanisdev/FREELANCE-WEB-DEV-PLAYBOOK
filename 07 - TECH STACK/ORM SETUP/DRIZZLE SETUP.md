# DRIZZLE SETUP [ LIGHTWEIGHT SQL ORM / EDGE-READY ]
------------------------------------------------------------------------

Drizzle is a lightweight, type-safe SQL toolkit. You write schemas in
TypeScript, queries read like SQL, and there is no heavy runtime — it
runs on the edge. A strong fit for Next.js App Router on serverless.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add drizzle-orm
pnpm add -D drizzle-kit
# driver for your database (Postgres example):
pnpm add postgres
```

For Neon serverless / edge, use `@neondatabase/serverless` instead of
`postgres`.

------------------------------------------------------------------------

## STEP 2 : Configure Environment

```bash
# .env.local
DATABASE_URL="postgresql://user:password@host.neon.tech/db?sslmode=require"
```

------------------------------------------------------------------------

## STEP 3 : Define The Schema

```ts
// db/schema.ts
import {
  pgTable, text, boolean, timestamp
} from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name"),
});

export const posts = pgTable("posts", {
  id: text("id").primaryKey(),
  title: text("title").notNull(),
  published: boolean("published").default(false),
  authorId: text("author_id").references(() => users.id),
  createdAt: timestamp("created_at").defaultNow(),
});
```

------------------------------------------------------------------------

## STEP 4 : Create The Client (Singleton)

```ts
// db/index.ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import * as schema from "./schema";

const globalForDb = globalThis as unknown as {
  client?: ReturnType<typeof postgres>;
};

const client =
  globalForDb.client ?? postgres(process.env.DATABASE_URL!);

if (process.env.NODE_ENV !== "production") globalForDb.client = client;

export const db = drizzle(client, { schema });
```

------------------------------------------------------------------------

## STEP 5 : drizzle-kit Migrations

Create a config, generate SQL from your schema, then push or migrate.

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: { url: process.env.DATABASE_URL! },
});
```

```bash
# generate SQL migration files from the schema
pnpm dlx drizzle-kit generate

# apply migrations to the database
pnpm dlx drizzle-kit migrate

# quick prototyping: push schema directly (no migration files)
pnpm dlx drizzle-kit push
```

```text
schema.ts  --generate-->  ./drizzle/*.sql  --migrate-->  database
                (versioned, commit these)
```

------------------------------------------------------------------------

## STEP 6 : Querying

```ts
// app/api/users/route.ts
import { db } from "@/db";
import { users } from "@/db/schema";
import { eq } from "drizzle-orm";

export async function GET() {
  const rows = await db.select().from(users).limit(10);
  return Response.json(rows);
}

export async function POST(req: Request) {
  const { id, email, name } = await req.json();
  const [row] = await db
    .insert(users)
    .values({ id, email, name })
    .returning();
  return Response.json(row, { status: 201 });
}

// relational query API
const withPosts = await db.query.users.findMany({
  with: { /* relations defined in schema */ },
});
```

------------------------------------------------------------------------

## WHEN TO CHOOSE DRIZZLE

- You deploy to the edge and need a tiny, cold-start-friendly runtime.
- You like SQL and want queries that mirror it, not a heavy abstraction.
- You want full type safety with minimal generated code.
- You prefer schema-as-TypeScript over a separate DSL file.

Choose Prisma instead if you want a richer client, Studio GUI, and a
more managed migration workflow out of the box.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Cache the underlying client on globalThis for hot reload
✓ Commit generated SQL migration files in ./drizzle
✓ Use generate + migrate for prod; reserve push for prototyping
✓ Pick the edge driver (neon) for edge runtime deployments
✓ Keep DATABASE_URL out of source control
✓ Define relations in schema to use the relational query API
✓ Use eq/and/or helpers instead of string-built WHERE clauses
✓ Run migrations in CI before the app boots
```
