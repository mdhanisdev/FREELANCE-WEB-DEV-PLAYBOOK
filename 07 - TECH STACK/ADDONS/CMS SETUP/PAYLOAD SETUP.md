# PAYLOAD SETUP [ CMS ]
------------------------------------------------------------------------
Payload is a self-hosted, code-first CMS that runs inside your own
Next.js app. It ships an admin UI, a REST + GraphQL API, and stores
content in your own Postgres database. You own the data end to end.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add payload @payloadcms/next @payloadcms/richtext-lexical
pnpm add @payloadcms/db-postgres
pnpm add -D @payloadcms/graphql
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
```text
DATABASE_URI=postgres://user:pass@localhost:5432/app
PAYLOAD_SECRET=generate_a_long_random_string
NEXT_PUBLIC_SERVER_URL=http://localhost:3000
```
`PAYLOAD_SECRET` signs JWTs and encrypts fields, keep it out of git.
------------------------------------------------------------------------
## STEP 3 : Payload Config
------------------------------------------------------------------------
```ts
// payload.config.ts
import { buildConfig } from "payload";
import { postgresAdapter } from "@payloadcms/db-postgres";
import { lexicalEditor } from "@payloadcms/richtext-lexical";

export default buildConfig({
  secret: process.env.PAYLOAD_SECRET!,
  editor: lexicalEditor(),
  db: postgresAdapter({ pool: { connectionString: process.env.DATABASE_URI! } }),
  collections: [
    { slug: "users", auth: true, fields: [{ name: "name", type: "text" }] },
    {
      slug: "posts",
      admin: { useAsTitle: "title" },
      fields: [
        { name: "title", type: "text", required: true },
        { name: "slug", type: "text", unique: true, index: true },
        { name: "content", type: "richText" },
        { name: "published", type: "checkbox", defaultValue: false },
      ],
    },
  ],
});
```
------------------------------------------------------------------------
## STEP 4 : Mount the Admin & API Routes
------------------------------------------------------------------------
Payload injects its own App Router segments. Wrap `next.config` and add
the admin route group.
```ts
// next.config.ts
import { withPayload } from "@payloadcms/next/withPayload";
export default withPayload({ /* your next config */ });
```
```text
src/app/
  (payload)/
    admin/[[...segments]]/page.tsx   → hosted admin UI at /admin
    api/[...slug]/route.ts           → REST + GraphQL endpoints
```
------------------------------------------------------------------------
## STEP 5 : Query Content in Server Components
------------------------------------------------------------------------
```ts
// src/lib/payload.ts
import { getPayload } from "payload";
import config from "../../payload.config";

export async function getPublishedPosts() {
  const payload = await getPayload({ config });
  const { docs } = await payload.find({
    collection: "posts",
    where: { published: { equals: true } },
    sort: "-createdAt",
  });
  return docs;
}
```
```tsx
// src/app/blog/page.tsx
import { getPublishedPosts } from "@/lib/payload";

export default async function Blog() {
  const posts = await getPublishedPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 6 : Migrate the Database
------------------------------------------------------------------------
```bash
pnpm payload migrate:create init
pnpm payload migrate
```
Run migrations in CI before deploy so schema and code stay in lockstep.
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
        ┌──────────── Single Next.js Deploy ────────────┐
        │  /admin  ── Payload Admin UI (editors)         │
        │  /api    ── REST + GraphQL                     │
        │  getPayload() ── Local API (no HTTP hop)       │
        └──────────────────┬─────────────────────────────┘
                           ▼
                    Postgres (your DB)
```
Note (Windows): use a local Postgres via Docker Desktop or
`winget install PostgreSQL.PostgreSQL`; ensure `DATABASE_URI` host is
`localhost` not a Unix socket path.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Prefer the Local API (getPayload) in server components to skip HTTP
✓ Commit generated migrations and run them in CI before deploying
✓ Keep PAYLOAD_SECRET stable across deploys or existing sessions break
✓ Use access-control functions per collection, not just UI hiding
✓ Index fields you filter or sort on (slug, publishedAt)
✓ Store media in S3/R2 via a storage adapter, not the local filesystem
✓ Enable drafts + versions for editorial review workflows
✓ Back up Postgres regularly; the CMS is only as safe as your DB
```
