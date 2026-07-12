# POSTGRESQL SETUP [ RELATIONAL DATABASE / MANAGED HOSTING ]
------------------------------------------------------------------------

PostgreSQL is a battle-tested relational database. For a Next.js App
Router deployment you almost never run it yourself in production — you
point a connection string at a managed host and let them handle
backups, failover, and scaling.

------------------------------------------------------------------------

## STEP 1 : Pick A Managed Host

- Neon (recommended) — serverless Postgres, scales to zero, instant
  branching per Git branch, generous free tier. Best fit for Vercel.
- Supabase — Postgres plus auth, storage, and realtime bundled in.
- Railway — simple provisioning, predictable pricing, good for
  always-on workloads.

```text
   Next.js (App Router)
          |
     DATABASE_URL  (pooled)
          |
   +--------------+       +----------------+
   |  Neon Proxy  | ----> |  Postgres node |
   | (pgBouncer)  |       |  (primary)     |
   +--------------+       +----------------+
```

------------------------------------------------------------------------

## STEP 2 : Get The Connection String

Create a project in the dashboard, then copy the pooled connection
string. It looks like this (placeholders only — never commit real
credentials):

```text
postgresql://user:password@ep-cool-name-123.us-east-2.aws.neon.tech/dbname?sslmode=require
```

Two variants matter:
- Pooled URL (`-pooler` host) — use for the app at runtime.
- Direct URL — use for migrations that need a single session.

------------------------------------------------------------------------

## STEP 3 : Store It In Environment Variables

Create `.env.local` at the project root. On Windows this is a plain
UTF-8 file — do not commit it.

```bash
# .env.local
DATABASE_URL="postgresql://user:password@ep-cool-name-123-pooler.us-east-2.aws.neon.tech/dbname?sslmode=require"
DIRECT_URL="postgresql://user:password@ep-cool-name-123.us-east-2.aws.neon.tech/dbname?sslmode=require"
```

`sslmode=require` forces an encrypted connection. Managed hosts reject
plaintext, so keep it on always.

------------------------------------------------------------------------

## STEP 4 : Install A Driver

For raw SQL or as a base for an ORM, install a driver. Neon ships an
edge-compatible serverless driver.

```bash
pnpm add pg
pnpm add -D @types/pg
# edge / serverless alternative:
pnpm add @neondatabase/serverless
```

------------------------------------------------------------------------

## STEP 5 : Singleton Connection Concept

Next.js in dev hot-reloads modules constantly. If you create a new
connection pool on every reload you exhaust the database's connection
limit. The fix is a single cached pool stored on `globalThis`.

```ts
// lib/db.ts
import { Pool } from "pg";

const globalForPg = globalThis as unknown as { pool?: Pool };

export const pool =
  globalForPg.pool ??
  new Pool({ connectionString: process.env.DATABASE_URL });

if (process.env.NODE_ENV !== "production") globalForPg.pool = pool;
```

```text
DEV (hot reload)              PRODUCTION
+------------------+          +------------------+
| reload #1 -> pool|          | one process      |
| reload #2 -> SAME|  <-cache | one pool         |
| reload #3 -> SAME|          | reused per req   |
+------------------+          +------------------+
```

------------------------------------------------------------------------

## STEP 6 : Query It

```ts
// app/api/users/route.ts
import { pool } from "@/lib/db";

export async function GET() {
  const { rows } = await pool.query(
    "SELECT id, email FROM users LIMIT 10"
  );
  return Response.json(rows);
}
```

------------------------------------------------------------------------

## WHEN TO USE POSTGRES

- Relational data with clear schema and foreign keys.
- Transactions and strong consistency (orders, payments, inventory).
- Complex queries, joins, aggregations, JSONB when you need it.
- Anything you would reach for SQL to answer.

Avoid when your data is truly schema-less document blobs (MongoDB) or
you only need ephemeral key/value caching (Redis).

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use the pooled connection string for the running app
✓ Use the direct URL for migrations needing one session
✓ Keep sslmode=require on every connection
✓ Cache one Pool on globalThis to survive hot reload
✓ Never commit .env.local — add it to .gitignore
✓ Set a sane pool max and statement timeout in production
✓ Prefer parameterized queries ($1, $2) to block SQL injection
✓ Enable branching (Neon) for isolated preview databases
✓ Store secrets in the host's env manager, not in code
```
