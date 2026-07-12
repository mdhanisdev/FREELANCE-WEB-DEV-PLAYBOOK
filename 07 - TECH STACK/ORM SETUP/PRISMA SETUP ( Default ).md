# PRISMA SETUP [ TYPE-SAFE ORM / SQL DATABASES ]
------------------------------------------------------------------------

Prisma is a type-safe ORM for Postgres, MySQL, and more. You describe
your data in a schema file, Prisma generates a fully typed client, and
migrations keep the database in sync. Great fit for Next.js App Router.

------------------------------------------------------------------------

## STEP 1 : Install And Initialize

```bash
pnpm add -D prisma
pnpm add @prisma/client
pnpm dlx prisma init
```

`prisma init` creates `prisma/schema.prisma` and adds `DATABASE_URL` to
`.env`. On Windows, run these from the project root in your terminal.

------------------------------------------------------------------------

## STEP 2 : Configure Environment

```bash
# .env
DATABASE_URL="postgresql://user:password@host-pooler.neon.tech/db?sslmode=require"
DIRECT_URL="postgresql://user:password@host.neon.tech/db?sslmode=require"
```

------------------------------------------------------------------------

## STEP 3 : Define The Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

model User {
  id    String  @id @default(cuid())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        String   @id @default(cuid())
  title     String
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}
```

Use the pooled `url` at runtime and the `directUrl` for migrations.

------------------------------------------------------------------------

## STEP 4 : Singleton Client

Next.js hot reload creates many `PrismaClient` instances in dev, which
exhausts DB connections. Cache one instance on `globalThis`.

```ts
// lib/prisma.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma?: PrismaClient;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}
```

------------------------------------------------------------------------

## STEP 5 : Migrate — Dev vs Deploy

```bash
# during development: create + apply a migration, regenerate client
pnpm dlx prisma migrate dev --name init

# in CI / production: apply committed migrations, no prompts
pnpm dlx prisma migrate deploy
```

```text
LOCAL DEV                         PRODUCTION / CI
+-----------------------+         +----------------------+
| migrate dev           |         | migrate deploy       |
| - detects schema drift|  --->   | - applies committed  |
| - writes new migration|  commit |   migrations only    |
| - regenerates client  |  files  | - no schema changes  |
+-----------------------+         +----------------------+
```

Never run `migrate dev` against production — it can prompt and reset.

------------------------------------------------------------------------

## STEP 6 : Generate Client In postinstall

On hosts like Vercel, dependencies are cached, so the generated client
can go stale. Regenerate it on every install.

```json
// package.json
{
  "scripts": {
    "postinstall": "prisma generate",
    "build": "prisma generate && next build"
  }
}
```

------------------------------------------------------------------------

## STEP 7 : Querying

```ts
// app/api/users/route.ts
import { prisma } from "@/lib/prisma";

export async function GET() {
  const users = await prisma.user.findMany({
    where: { posts: { some: { published: true } } },
    include: { posts: true },
    take: 10,
  });
  return Response.json(users);
}

export async function POST(req: Request) {
  const { email, name } = await req.json();
  const user = await prisma.user.create({ data: { email, name } });
  return Response.json(user, { status: 201 });
}
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Cache one PrismaClient on globalThis to survive hot reload
✓ Use migrate dev locally, migrate deploy in CI/production
✓ Commit the prisma/migrations folder to version control
✓ Add prisma generate to postinstall so the client stays fresh
✓ Use directUrl for migrations, pooled url for the running app
✓ Keep DATABASE_URL secret — never commit real credentials
✓ Use select/include to fetch only the fields you need
✓ Prefer transactions (prisma.$transaction) for multi-step writes
```
