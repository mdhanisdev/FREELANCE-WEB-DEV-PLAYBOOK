# SANITY SETUP [ CMS ]
------------------------------------------------------------------------
Sanity is the default CMS: a hosted, real-time content backend with a
fully embeddable Studio, structured content, and the GROQ query language.
Non-technical editors get a polished UI; developers get typed content.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add next-sanity @sanity/image-url @sanity/vision sanity styled-components
pnpm add -D @sanity/types
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables
------------------------------------------------------------------------
Create a project at sanity.io/manage and copy the IDs into `.env.local`.
```text
NEXT_PUBLIC_SANITY_PROJECT_ID=xxxxxxxx
NEXT_PUBLIC_SANITY_DATASET=production
NEXT_PUBLIC_SANITY_API_VERSION=2024-10-01
SANITY_API_READ_TOKEN=sk_live_xxxxxxxxxxxx
```
Never expose write tokens to the client. Only the read token is used in
server components; keep it out of `NEXT_PUBLIC_*`.
------------------------------------------------------------------------
## STEP 3 : Client & Config
------------------------------------------------------------------------
```ts
// src/sanity/env.ts
export const apiVersion = process.env.NEXT_PUBLIC_SANITY_API_VERSION!;
export const dataset = process.env.NEXT_PUBLIC_SANITY_DATASET!;
export const projectId = process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!;
```
```ts
// src/sanity/client.ts
import { createClient } from "next-sanity";
import { apiVersion, dataset, projectId } from "./env";

export const client = createClient({
  projectId,
  dataset,
  apiVersion,
  useCdn: true, // false when you need fresh data on every request
});
```
------------------------------------------------------------------------
## STEP 4 : Define a Schema
------------------------------------------------------------------------
```ts
// src/sanity/schemas/post.ts
import { defineType, defineField } from "sanity";

export const post = defineType({
  name: "post",
  title: "Post",
  type: "document",
  fields: [
    defineField({ name: "title", type: "string", validation: (r) => r.required() }),
    defineField({ name: "slug", type: "slug", options: { source: "title" } }),
    defineField({ name: "coverImage", type: "image", options: { hotspot: true } }),
    defineField({ name: "body", type: "array", of: [{ type: "block" }] }),
    defineField({ name: "publishedAt", type: "datetime" }),
  ],
});
```
------------------------------------------------------------------------
## STEP 5 : Embed the Studio
------------------------------------------------------------------------
```ts
// sanity.config.ts
import { defineConfig } from "sanity";
import { structureTool } from "sanity/structure";
import { visionTool } from "@sanity/vision";
import { post } from "./src/sanity/schemas/post";
import { apiVersion, dataset, projectId } from "./src/sanity/env";

export default defineConfig({
  basePath: "/studio",
  projectId,
  dataset,
  schema: { types: [post] },
  plugins: [structureTool(), visionTool({ defaultApiVersion: apiVersion })],
});
```
```tsx
// src/app/studio/[[...tool]]/page.tsx
import { NextStudio } from "next-sanity/studio";
import config from "../../../../sanity.config";

export const dynamic = "force-static";
export default function StudioPage() {
  return <NextStudio config={config} />;
}
```
Editors now log in at `/studio` and publish content live.
------------------------------------------------------------------------
## STEP 6 : Query with GROQ
------------------------------------------------------------------------
```ts
// src/sanity/queries.ts
import { client } from "./client";

export async function getPosts() {
  return client.fetch<Array<{ title: string; slug: string }>>(
    `*[_type == "post"] | order(publishedAt desc){
       title, "slug": slug.current
     }`
  );
}
```
```tsx
// src/app/blog/page.tsx
import { getPosts } from "@/sanity/queries";

export const revalidate = 60; // ISR: refresh content every 60s

export default async function Blog() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.slug}>{p.title}</li>)}</ul>;
}
```
------------------------------------------------------------------------
## STEP 7 : Architecture
------------------------------------------------------------------------
```text
   Editor ── /studio (embedded) ──┐
                                  ▼
                          Sanity Content Lake
                                  │  GROQ + CDN
                                  ▼
   Next.js Server Component ── client.fetch() ── ISR cache ── Browser
```
Note (Windows): run `pnpm dlx sanity login` in PowerShell or Git Bash;
the CLI stores auth under `%USERPROFILE%\.config\sanity`.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep write tokens server-only; expose only the read token via NEXT_PUBLIC
✓ Use useCdn:true for reads, false only for preview/draft mode
✓ Pin apiVersion to a date string so query behavior never shifts silently
✓ Model reusable blocks as references, not duplicated inline objects
✓ Enable hotspot on images and generate URLs via @sanity/image-url
✓ Use ISR (revalidate) or webhooks over on-demand fetch for scale
✓ Validate required fields in schema so editors cannot publish bad data
✓ Gate /studio behind Sanity auth; never ship write tokens to the client
```
