# MDX SETUP [ CMS ]
------------------------------------------------------------------------
MDX is a dev-owned content approach: Markdown with embedded JSX lives in
your repo, is version-controlled, and compiles at build time. No backend,
no editor UI. Ideal for docs, changelogs, and engineering blogs.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
------------------------------------------------------------------------
```bash
pnpm add next-mdx-remote gray-matter
pnpm add rehype-slug rehype-pretty-code remark-gfm
```
`next-mdx-remote` compiles MDX from the filesystem; `gray-matter` parses
frontmatter; the rehype/remark plugins add slugs, GFM, and code themes.
------------------------------------------------------------------------
## STEP 2 : Content Layout
------------------------------------------------------------------------
```text
content/
  posts/
    hello-world.mdx
    shipping-fast.mdx
src/app/blog/[slug]/page.tsx
src/lib/mdx.ts
```
```text
---
title: Hello World
publishedAt: 2026-07-01
summary: The first post.
---

# Hello World

This is **MDX** with an embedded <Callout>component</Callout>.
```
------------------------------------------------------------------------
## STEP 3 : Content Loader
------------------------------------------------------------------------
```ts
// src/lib/mdx.ts
import fs from "node:fs/promises";
import path from "node:path";
import matter from "gray-matter";

const DIR = path.join(process.cwd(), "content", "posts");

export interface PostMeta {
  title: string;
  publishedAt: string;
  summary: string;
  slug: string;
}

export async function getSlugs() {
  const files = await fs.readdir(DIR);
  return files.filter((f) => f.endsWith(".mdx")).map((f) => f.replace(/\.mdx$/, ""));
}

export async function getPost(slug: string) {
  const raw = await fs.readFile(path.join(DIR, `${slug}.mdx`), "utf8");
  const { content, data } = matter(raw);
  return { content, meta: { ...data, slug } as PostMeta };
}
```
------------------------------------------------------------------------
## STEP 4 : Render with Custom Components
------------------------------------------------------------------------
```tsx
// src/app/blog/[slug]/page.tsx
import { MDXRemote } from "next-mdx-remote/rsc";
import rehypeSlug from "rehype-slug";
import rehypePrettyCode from "rehype-pretty-code";
import remarkGfm from "remark-gfm";
import { getPost, getSlugs } from "@/lib/mdx";

const components = {
  Callout: (p: { children: React.ReactNode }) => (
    <aside className="callout">{p.children}</aside>
  ),
};

export async function generateStaticParams() {
  return (await getSlugs()).map((slug) => ({ slug }));
}

export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const { content } = await getPost(slug);
  return (
    <article>
      <MDXRemote
        source={content}
        components={components}
        options={{
          mdxOptions: {
            remarkPlugins: [remarkGfm],
            rehypePlugins: [rehypeSlug, rehypePrettyCode],
          },
        }}
      />
    </article>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Index Page
------------------------------------------------------------------------
```tsx
// src/app/blog/page.tsx
import Link from "next/link";
import { getSlugs, getPost } from "@/lib/mdx";

export default async function Blog() {
  const slugs = await getSlugs();
  const posts = await Promise.all(slugs.map((s) => getPost(s)));
  posts.sort((a, b) => b.meta.publishedAt.localeCompare(a.meta.publishedAt));
  return (
    <ul>
      {posts.map(({ meta }) => (
        <li key={meta.slug}>
          <Link href={`/blog/${meta.slug}`}>{meta.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Architecture
------------------------------------------------------------------------
```text
   Dev writes .mdx ──► git commit ──► CI build
                                        │
                       generateStaticParams() enumerates slugs
                                        │
                          MDXRemote compiles at build time
                                        ▼
                       Static HTML (fully prerendered) ── Browser
```
Note (Windows): always build paths with `node:path`; never hard-code
`/` separators, and read files with `node:fs/promises` for portability.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Treat content as code: PR review, lint, and version every change
✓ Prerender with generateStaticParams so pages are fully static
✓ Validate frontmatter shape (zod) at build to catch typos early
✓ Use node:path.join, never string-concatenate file paths
✓ Whitelist MDX components; do not eval untrusted MDX from users
✓ Add rehype-slug + heading anchors for linkable sections
✓ Keep content/ separate from src/ so authoring stays obvious
✓ Cache getPost during build; avoid re-reading files per request
```
