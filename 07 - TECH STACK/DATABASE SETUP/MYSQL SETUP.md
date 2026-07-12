# MYSQL SETUP [ RELATIONAL DATABASE / MANAGED HOSTING ]
------------------------------------------------------------------------

MySQL is a widely supported relational database. For Next.js App Router
apps the ergonomic path is a managed, serverless-friendly host that
speaks the MySQL protocol over HTTP or a connection pool.

------------------------------------------------------------------------

## STEP 1 : Pick A Managed Host

- PlanetScale — serverless MySQL built on Vitess, non-blocking schema
  changes via branches and deploy requests, HTTP driver for the edge.
- Railway MySQL — simple always-on instance, good for small apps.
- AWS RDS / Aurora MySQL — enterprise scale and control.

```text
   Next.js (App Router)
          |
     DATABASE_URL
          |
   +----------------+      +------------------+
   |  PlanetScale   | ---> |  Vitess shards   |
   |  HTTP / pool   |      |  (MySQL nodes)   |
   +----------------+      +------------------+
```

------------------------------------------------------------------------

## STEP 2 : Get The Connection String

Create a database and a branch (PlanetScale defaults to `main`), then
generate a password. The connection string uses TLS:

```text
mysql://user:password@aws.connect.psdb.cloud/dbname?ssl={"rejectUnauthorized":true}
```

For a classic host it is simpler:

```text
mysql://user:password@host.example.com:3306/dbname?sslaccept=strict
```

------------------------------------------------------------------------

## STEP 3 : Store It In Environment Variables

```bash
# .env.local
DATABASE_URL="mysql://user:password@aws.connect.psdb.cloud/dbname?sslaccept=strict"
```

TLS is mandatory on managed MySQL. `sslaccept=strict` verifies the
server certificate. Keep `.env.local` out of version control.

------------------------------------------------------------------------

## STEP 4 : Install A Driver

The standard pooled driver is `mysql2`. For the edge, PlanetScale
provides a fetch-based serverless driver.

```bash
pnpm add mysql2
# edge / serverless (HTTP) alternative:
pnpm add @planetscale/database
```

------------------------------------------------------------------------

## STEP 5 : Singleton Connection Pool

As with any database in Next.js, cache one pool on `globalThis` so hot
reload in dev does not open hundreds of connections.

```ts
// lib/db.ts
import mysql from "mysql2/promise";

const globalForDb = globalThis as unknown as {
  pool?: mysql.Pool;
};

export const pool =
  globalForDb.pool ??
  mysql.createPool({ uri: process.env.DATABASE_URL, ssl: {} });

if (process.env.NODE_ENV !== "production") globalForDb.pool = pool;
```

```text
+------------------------------------------------+
| one Pool  ->  N reusable connections           |
| request -> borrow conn -> query -> return conn |
+------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 6 : Query It

```ts
// app/api/products/route.ts
import { pool } from "@/lib/db";

export async function GET() {
  const [rows] = await pool.query(
    "SELECT id, name FROM products LIMIT 10"
  );
  return Response.json(rows);
}
```

------------------------------------------------------------------------

## WHEN TO USE MYSQL

- Relational data where your team already knows MySQL.
- Read-heavy workloads that benefit from Vitess horizontal scaling.
- You want online, non-blocking schema migrations (PlanetScale).
- Existing tooling, ORMs, and hosting expect MySQL.

Note: PlanetScale does not enforce foreign key constraints at the
Vitess layer — model relations in the application / ORM instead. If you
need enforced FKs and rich SQL features, prefer Postgres.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always connect over TLS (sslaccept=strict)
✓ Cache one connection pool on globalThis
✓ Use branches + deploy requests for safe schema changes
✓ Use parameterized queries (?) to prevent SQL injection
✓ Keep the HTTP driver for edge runtime, mysql2 for Node
✓ Never commit credentials — use the host env manager
✓ Model relations in the ORM since FKs may be unenforced
✓ Set connectionLimit to match your host's plan
```
