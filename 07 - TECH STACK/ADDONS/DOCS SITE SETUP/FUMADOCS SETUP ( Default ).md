# FUMADOCS SETUP [ DOCS SITE ]

------------------------------------------------------------------------

Fumadocs is a Next.js-native documentation framework. Because it runs
inside the App Router itself, you reuse your existing components, styling,
and deployment — no separate site to maintain. This is the default docs
engine for the stack.

------------------------------------------------------------------------

## STEP 1 : Scaffold Into An App

Add Fumadocs to an existing Next.js project:

```bash
pnpm add fumadocs-ui fumadocs-core fumadocs-mdx
pnpm add -D @types/mdx
```

Or start fresh with the official initializer:

```bash
pnpm create fumadocs-app
```

------------------------------------------------------------------------

## STEP 2 : Configure The MDX Source

```ts
// source.config.ts
import { defineDocs, defineConfig } from "fumadocs-mdx/config";

export const docs = defineDocs({
  dir: "content/docs",
});

export default defineConfig({
  mdxOptions: {
    // rehype/remark plugins go here
  },
});
```

```ts
// next.config.ts
import { createMDX } from "fumadocs-mdx/next";

const withMDX = createMDX();
export default withMDX({ reactStrictMode: true });
```

------------------------------------------------------------------------

## STEP 3 : Wire The Source Adapter

```ts
// lib/source.ts
import { docs } from "@/.source";
import { loader } from "fumadocs-core/source";

export const source = loader({
  baseUrl: "/docs",
  source: docs.toFumaSource(),
});
```

The `.source` folder is generated at build time from your `content/docs`
tree, giving you typed page and tree data.

------------------------------------------------------------------------

## STEP 4 : Route Layout And Pages

```text
   app/
   └── docs/
       ├── layout.tsx          → <DocsLayout> sidebar + nav
       └── [[...slug]]/
           └── page.tsx        → renders MDX by slug

   content/
   └── docs/
       ├── index.mdx           → /docs
       ├── meta.json           → sidebar order
       └── guides/
           └── quickstart.mdx  → /docs/guides/quickstart
```

```tsx
// app/docs/[[...slug]]/page.tsx
import { source } from "@/lib/source";
import { notFound } from "next/navigation";
import { DocsPage, DocsBody } from "fumadocs-ui/page";

export default async function Page(props: {
  params: Promise<{ slug?: string[] }>;
}) {
  const { slug } = await props.params;
  const page = source.getPage(slug);
  if (!page) notFound();

  const MDX = page.data.body;
  return (
    <DocsPage toc={page.data.toc}>
      <DocsBody>
        <MDX />
      </DocsBody>
    </DocsPage>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Author Content

```mdx
---
title: Quickstart
description: Get running in five minutes.
---

## Install

Use the package manager you already run.

<Callout type="info">Fumadocs ships MDX components out of the box.</Callout>
```

------------------------------------------------------------------------

## STEP 6 : Run And Deploy

```bash
pnpm dev      # docs live at /docs
pnpm build    # static + RSC output, deploy to Vercel as usual
```

On Windows, the generated `.source` directory uses forward-slash imports;
add it to `.gitignore` so line-ending diffs never appear in commits.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep docs in the same repo so they deploy with the app
✓ Order the sidebar explicitly with meta.json per folder
✓ Reuse app components inside MDX for living examples
✓ Gitignore the generated .source folder
✓ Add frontmatter title + description to every page for SEO
✓ Use built-in Callout and Tabs instead of custom HTML
✓ Enable full-text search via fumadocs-core search API
✓ Version docs by folder when you ship breaking changes
✓ Run pnpm build in CI to catch broken MDX links early
```
