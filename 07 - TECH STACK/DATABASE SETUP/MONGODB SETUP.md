# MONGODB SETUP [ DOCUMENT DATABASE / MANAGED HOSTING ]
------------------------------------------------------------------------

MongoDB stores flexible JSON-like documents instead of rows and tables.
For Next.js App Router apps, MongoDB Atlas is the managed host — it
handles clusters, backups, and network security for you.

------------------------------------------------------------------------

## STEP 1 : Create An Atlas Cluster

- Sign up at MongoDB Atlas and create a free M0 cluster.
- Create a database user with a strong password.
- Add a network access rule. For Vercel/serverless, allow `0.0.0.0/0`
  and rely on user credentials + TLS, or use Atlas private endpoints.

```text
   Next.js (App Router)
          |
   MONGODB_URI (mongodb+srv)
          |
   +---------------------+
   |  Atlas Cluster      |
   |  primary + replicas |
   +---------------------+
```

------------------------------------------------------------------------

## STEP 2 : Get The Connection URI

Click "Connect" then "Drivers" and copy the SRV URI (placeholders
only):

```text
mongodb+srv://user:password@cluster0.abcde.mongodb.net/dbname?retryWrites=true&w=majority
```

- `mongodb+srv://` resolves replica hosts via DNS automatically.
- `retryWrites=true` retries transient write failures.
- `w=majority` waits for a majority of nodes to acknowledge writes.

------------------------------------------------------------------------

## STEP 3 : Store It In Environment Variables

```bash
# .env.local
MONGODB_URI="mongodb+srv://user:password@cluster0.abcde.mongodb.net/dbname?retryWrites=true&w=majority"
```

TLS is on by default with `mongodb+srv`. Keep `.env.local` untracked.

------------------------------------------------------------------------

## STEP 4 : Install The Driver

```bash
pnpm add mongodb
```

------------------------------------------------------------------------

## STEP 5 : Singleton Client Concept

Opening a new `MongoClient` on every request (or every dev hot reload)
floods the cluster with connections. Create the client once and cache
the connect promise on `globalThis`.

```ts
// lib/mongodb.ts
import { MongoClient } from "mongodb";

const uri = process.env.MONGODB_URI!;
const globalForMongo = globalThis as unknown as {
  clientPromise?: Promise<MongoClient>;
};

const client = new MongoClient(uri);

export const clientPromise =
  globalForMongo.clientPromise ?? client.connect();

if (process.env.NODE_ENV !== "production") {
  globalForMongo.clientPromise = clientPromise;
}
```

```text
+---------------------------------------------+
| new MongoClient(uri)                        |
|        |                                    |
|   .connect()  -> Promise cached once        |
|        |                                    |
|   every request awaits the SAME promise     |
+---------------------------------------------+
```

------------------------------------------------------------------------

## STEP 6 : Query It

```ts
// app/api/posts/route.ts
import { clientPromise } from "@/lib/mongodb";

export async function GET() {
  const client = await clientPromise;
  const db = client.db("dbname");
  const posts = await db
    .collection("posts")
    .find({})
    .limit(10)
    .toArray();
  return Response.json(posts);
}
```

------------------------------------------------------------------------

## WHEN TO USE MONGODB (DOCUMENT MODEL)

- Data shaped like nested JSON documents that vary between records.
- Schema evolves fast and you do not want migrations for every change.
- Content, catalogs, events, user profiles with embedded sub-objects.
- Horizontal scale via sharding on large collections.

Avoid when your data is highly relational with many joins and strict
multi-table transactions — a SQL database (Postgres) fits better.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Reuse one MongoClient — cache the connect promise on globalThis
✓ Keep mongodb+srv so TLS and replica discovery stay on
✓ Use w=majority for durable, acknowledged writes
✓ Index the fields you query and sort on
✓ Store credentials in env vars, never in source
✓ Prefer embedding related data over deep cross-collection joins
✓ Use a scoped database user, not the Atlas admin account
✓ Validate document shape at the app layer (or with Mongoose)
```
