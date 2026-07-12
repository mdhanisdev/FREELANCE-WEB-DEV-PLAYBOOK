# STRAPI SETUP [ CMS ]
------------------------------------------------------------------------
Strapi is an open-source, self-hosted headless CMS with a customizable
admin panel and an auto-generated REST + GraphQL API. It runs as a
separate service that your Next.js frontend consumes over HTTP.
------------------------------------------------------------------------
## STEP 1 : Scaffold the Strapi Backend
------------------------------------------------------------------------
Strapi runs in its own directory, separate from the Next.js app.
```bash
pnpm dlx create-strapi-app@latest cms --quickstart
cd cms && pnpm develop
```
The admin panel opens at `http://localhost:1337/admin` for first-run
setup. In the frontend app, add the SDK:
```bash
pnpm add @strapi/client
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Generate a read-only API token in Settings → API Tokens.
```text
NEXT_PUBLIC_STRAPI_URL=http://localhost:1337
STRAPI_API_TOKEN=xxxxxxxxxxxxxxxxxxxx
```
------------------------------------------------------------------------
## STEP 3 : Define a Content Type
------------------------------------------------------------------------
Use the Content-Type Builder in the admin panel to create `Post` with
fields `title` (Text), `slug` (UID), `body` (Rich Text), then set
Public/Authenticated permissions under Settings → Roles.
```text
Content-Type Builder → Create new collection type → "Post"
  + title  : Text (required)
  + slug   : UID (attached to title)
  + body   : Rich Text (Blocks)
  + cover  : Media (single)
```
------------------------------------------------------------------------
## STEP 4 : Client
------------------------------------------------------------------------
```ts
// src/lib/strapi.ts
import { strapi } from "@strapi/client";

export const cms = strapi({
  baseURL: `${process.env.NEXT_PUBLIC_STRAPI_URL}/api`,
  auth: process.env.STRAPI_API_TOKEN,
});
```
------------------------------------------------------------------------
## STEP 5 : Fetch Content
------------------------------------------------------------------------
```ts
// src/lib/queries.ts
import { cms } from "./strapi";

export async function getPosts() {
  const posts = cms.collection("posts");
  const { data } = await posts.find({
    sort: ["publishedAt:desc"],
    populate: ["cover"],
    pagination: { pageSize: 20 },
  });
  return data as Array<{ id: number; title: string; slug: string }>;
}
```
```tsx
// src/app/blog/page.tsx
import { getPosts } from "@/lib/queries";

export const revalidate = 60;

export default async function Blog() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 6 : Deploy Considerations
------------------------------------------------------------------------
Strapi needs a persistent Node host (Railway, Render, Fly, a VPS) and a
managed Postgres in production, not the SQLite quickstart DB.
```bash
# production build of the CMS service
cd cms && pnpm build && pnpm start
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   Editor ── Strapi Admin (:1337) ──┐
                                    ▼
                      Strapi Server (Node)
                      REST /api + GraphQL
                            │
                            ▼
                    Postgres (prod DB)
                            │  HTTP + Bearer token
                            ▼
                  Next.js (ISR) ── Browser
```
Note (Windows): the quickstart uses SQLite which needs build tools;
if `better-sqlite3` fails, install
`winget install Microsoft.VisualStudio.2022.BuildTools` or switch to
Postgres locally.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Use scoped read-only API tokens for the frontend; never the admin one
✓ Move off SQLite to Postgres before production
✓ Set explicit Roles & Permissions; default-deny public write access
✓ Use populate deliberately, deep-populate only what you render
✓ Enable ISR and revalidate via lifecycle webhooks on publish
✓ Keep the CMS on its own deploy so admin traffic never hits the frontend
✓ Version content types via config sync, not manual clicking in prod
✓ Put media on S3/Cloudinary through a provider plugin, not local disk
```
