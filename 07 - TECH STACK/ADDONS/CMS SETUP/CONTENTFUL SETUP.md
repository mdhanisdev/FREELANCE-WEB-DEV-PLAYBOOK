# CONTENTFUL SETUP [ CMS ]
------------------------------------------------------------------------
Contentful is a hosted, API-first headless CMS with a mature web app for
editors, global CDN delivery, and separate Delivery (published) and
Preview (draft) APIs. Content models are defined in the dashboard.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add contentful @contentful/rich-text-react-renderer
pnpm add -D contentful-management contentful-cli
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create a Space at app.contentful.com and generate API keys.
```text
CONTENTFUL_SPACE_ID=xxxxxxxxxxxx
CONTENTFUL_ENVIRONMENT=master
CONTENTFUL_DELIVERY_TOKEN=cda_xxxxxxxxxxxx
CONTENTFUL_PREVIEW_TOKEN=cpa_xxxxxxxxxxxx
```
Delivery token serves published content; preview token serves drafts.
------------------------------------------------------------------------
## STEP 3 : Client
------------------------------------------------------------------------
```ts
// src/lib/contentful.ts
import { createClient } from "contentful";

const base = {
  space: process.env.CONTENTFUL_SPACE_ID!,
  environment: process.env.CONTENTFUL_ENVIRONMENT ?? "master",
};

export const cdaClient = createClient({
  ...base,
  accessToken: process.env.CONTENTFUL_DELIVERY_TOKEN!,
});

export const previewClient = createClient({
  ...base,
  host: "preview.contentful.com",
  accessToken: process.env.CONTENTFUL_PREVIEW_TOKEN!,
});

export const getClient = (preview = false) =>
  preview ? previewClient : cdaClient;
```
------------------------------------------------------------------------
## STEP 4 : Type Your Content Model
------------------------------------------------------------------------
Define the `post` content type in the web app, then mirror its shape.
```ts
// src/types/contentful.ts
import type { EntryFieldTypes } from "contentful";

export interface PostSkeleton {
  contentTypeId: "post";
  fields: {
    title: EntryFieldTypes.Text;
    slug: EntryFieldTypes.Text;
    body: EntryFieldTypes.RichText;
    publishedAt: EntryFieldTypes.Date;
  };
}
```
------------------------------------------------------------------------
## STEP 5 : Fetch Entries
------------------------------------------------------------------------
```ts
// src/lib/queries.ts
import { getClient } from "./contentful";
import type { PostSkeleton } from "@/types/contentful";

export async function getPosts(preview = false) {
  const res = await getClient(preview).getEntries<PostSkeleton>({
    content_type: "post",
    order: ["-fields.publishedAt"],
  });
  return res.items;
}
```
```tsx
// src/app/blog/page.tsx
import { getPosts } from "@/lib/queries";

export const revalidate = 120; // ISR revalidation window

export default async function Blog() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.sys.id}>{p.fields.title}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 6 : On-Demand Revalidation Webhook
------------------------------------------------------------------------
```ts
// src/app/api/revalidate/route.ts
import { revalidatePath } from "next/cache";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const secret = new URL(req.url).searchParams.get("secret");
  if (secret !== process.env.CONTENTFUL_WEBHOOK_SECRET)
    return NextResponse.json({ ok: false }, { status: 401 });
  revalidatePath("/blog");
  return NextResponse.json({ revalidated: true });
}
```
Point a Contentful webhook (on publish/unpublish) at this route.
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   Editor ── Contentful Web App ──► Publish
                                      │
              ┌───────────────────────┴───────────┐
              ▼ (published)                        ▼ (draft)
        Delivery API (CDN)                   Preview API
              │                                    │
              └──────► Next.js (ISR) ◄─────────────┘
                          │  webhook → revalidatePath
                          ▼
                       Browser
```
Note (Windows): install the CLI globally with
`pnpm add -g contentful-cli`; login token is cached in
`%USERPROFILE%\.contentfulrc.json`.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep delivery and preview tokens separate; never ship preview to prod
✓ Use ISR + publish webhooks instead of no-cache fetches
✓ Type entries with skeletons so field access is checked at compile time
✓ Model links as references and use include depth to hydrate in one call
✓ Use environments (master/staging) to test model changes safely
✓ Render RichText through the official renderer, never dangerouslySetInnerHTML
✓ Store the webhook secret server-side and verify it on every call
✓ Watch API rate limits; batch queries and cache aggressively
```
