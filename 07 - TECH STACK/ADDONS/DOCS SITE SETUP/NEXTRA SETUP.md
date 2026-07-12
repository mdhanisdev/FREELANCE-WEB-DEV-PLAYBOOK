# NEXTRA SETUP [ DOCS SITE ]

------------------------------------------------------------------------

Nextra is a batteries-included docs and blog theme built on Next.js. It
trades Fumadocs' component flexibility for convention: drop in MDX files
and get a polished site with search, dark mode, and nav for free. Use it
when you want docs shipped in an afternoon.

------------------------------------------------------------------------

## STEP 1 : Install Nextra

```bash
pnpm add nextra nextra-theme-docs
```

Nextra 4 targets the App Router. Keep it in its own Next.js app or a
monorepo package so its theme config never collides with your product.

------------------------------------------------------------------------

## STEP 2 : Configure next.config

```ts
// next.config.ts
import nextra from "nextra";

const withNextra = nextra({
  // search + syntax highlighting are enabled by default
});

export default withNextra({
  reactStrictMode: true,
});
```

------------------------------------------------------------------------

## STEP 3 : Add The Root Layout

```tsx
// app/layout.tsx
import { Layout, Navbar } from "nextra-theme-docs";
import { getPageMap } from "nextra/page-map";
import "nextra-theme-docs/style.css";

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <Layout navbar={<Navbar logo={<b>Docs</b>} />} pageMap={await getPageMap()}>
          {children}
        </Layout>
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Author Content With MDX

```text
   content/
   ├── index.mdx          → /
   ├── _meta.ts           → nav order + titles
   └── guides/
       ├── _meta.ts
       ├── install.mdx     → /guides/install
       └── deploy.mdx      → /guides/deploy
```

```ts
// content/_meta.ts
export default {
  index: "Introduction",
  guides: "Guides",
};
```

```mdx
---
title: Install
---

# Install

Run the install command and start the dev server.
```

------------------------------------------------------------------------

## STEP 5 : Run The Site

```bash
pnpm dev      # http://localhost:3000
pnpm build && pnpm start
```

Full-text search (Pagefind) indexes at build time — you must run
`pnpm build` at least once for search to work locally.

------------------------------------------------------------------------

## STEP 6 : Deploy

```text
   MDX files ──▶ Nextra theme ──▶ Next.js build ──▶ Vercel/static host
        │              │                │
     _meta.ts      dark mode +       Pagefind search
     ordering      TOC + nav          index bundled
```

Deploy to Vercel with zero extra config; the theme is fully static-safe.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use _meta.ts files to control nav order and labels
✓ Keep Nextra in its own app/package to isolate theme CSS
✓ Run a full build so Pagefind search indexes content
✓ Prefer Nextra over Fumadocs for speed, Fumadocs for control
✓ Add frontmatter titles for clean tab and SEO output
✓ Enable suppressHydrationWarning for the dark-mode toggle
✓ Lean on built-in Callout, Tabs, and Steps components
✓ Commit content as MDX so diffs stay reviewable
✓ Pin the nextra + theme versions together on upgrades
```
