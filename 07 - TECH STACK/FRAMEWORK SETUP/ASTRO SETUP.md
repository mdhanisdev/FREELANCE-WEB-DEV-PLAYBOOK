# ASTRO SETUP [ CONTENT & MARKETING SITES ]
------------------------------------------------------------------------
## STEP 1 : Scaffold The Project

Astro ships zero JS by default — ideal for content-heavy sites.

```bash
pnpm create astro@latest my-site
cd my-site
pnpm dev
```

Choose the "Empty" or "Blog" template and "Strict" TypeScript when
prompted. Windows note: the wizard uses arrow keys; run it in a real
terminal (Windows Terminal / Git Bash), not a non-interactive shell.
------------------------------------------------------------------------
## STEP 2 : Add Integrations

Use the `astro add` command — it installs and wires config automatically.

```bash
pnpm astro add tailwind
pnpm astro add react
pnpm astro add sitemap
```

Set `site` so the sitemap emits absolute URLs.

```ts
// astro.config.mjs
import { defineConfig } from "astro/config";
import sitemap from "@astrojs/sitemap";
import react from "@astrojs/react";

export default defineConfig({
  site: "https://www.example.com",
  integrations: [react(), sitemap()],
});
```

Tailwind v4 is wired via the Vite plugin in `astro add tailwind`.
------------------------------------------------------------------------
## STEP 3 : Content Collections

Type-safe Markdown/MDX with schema validation via Zod.

```ts
// src/content.config.ts
import { defineCollection, z } from "astro:content";
import { glob } from "astro/loaders";

const blog = defineCollection({
  loader: glob({ pattern: "**/*.md", base: "./src/content/blog" }),
  schema: z.object({
    title: z.string(),
    pubDate: z.coerce.date(),
    draft: z.boolean().default(false),
  }),
});

export const collections = { blog };
```

```text
src/content/blog/
├── hello-world.md
└── shipping-fast.md
```

Query entries with `getCollection("blog")`; invalid frontmatter fails the
build, so bad content never reaches production.
------------------------------------------------------------------------
## STEP 4 : Render A Collection

```astro
---
// src/pages/blog/index.astro
import { getCollection } from "astro:content";
const posts = (await getCollection("blog"))
  .filter((p) => !p.data.draft)
  .sort((a, b) => +b.data.pubDate - +a.data.pubDate);
---
<ul>
  {posts.map((post) => (
    <li><a href={`/blog/${post.id}`}>{post.data.title}</a></li>
  ))}
</ul>
```
------------------------------------------------------------------------
## STEP 5 : Islands — Hydrate Only What Moves

Static HTML by default; opt specific components into interactivity.

```astro
---
import Counter from "../components/Counter.tsx";
import Newsletter from "../components/Newsletter.tsx";
---
<Counter client:visible />        <!-- hydrate when scrolled into view -->
<Newsletter client:idle />        <!-- hydrate when the browser is idle -->
```

```text
Page HTML (0 KB JS)
   │
   ├─ <Counter client:visible> ─► JS loads only near viewport
   └─ <Newsletter client:idle> ─► JS loads after initial paint
```

Directives: `client:load` (immediate), `client:idle`, `client:visible`,
`client:media="(min-width:768px)"`. Prefer `visible`/`idle` for perf.
------------------------------------------------------------------------
## STEP 6 : Build & Preview

```bash
pnpm build      # static output in dist/ by default
pnpm preview    # serve the production build locally
```

Add an SSR adapter (`pnpm astro add node` / `vercel`) only if you need
server rendering; static output is the default and fastest path.
------------------------------------------------------------------------
## STEP 7 : When To Use Astro vs Next.js

Choose Astro for blogs, docs, and marketing pages where content and Core
Web Vitals dominate and interactivity is sparse — its island model ships
near-zero JS. Choose Next.js for app-like products: dashboards, auth,
heavy client state, and RSC data flows. Rule of thumb: mostly-read pages
→ Astro; mostly-interact app → Next.js.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep pages static; hydrate only interactive islands
✓ Prefer client:visible / client:idle over client:load
✓ Define Zod schemas so invalid content fails the build
✓ Set `site` in config for correct sitemap + canonical URLs
✓ Use astro add to wire integrations instead of manual config
✓ Author content in Markdown/MDX under src/content, not in pages
✓ Add an SSR adapter only when a page genuinely needs the server
✓ Audit bundle size — every island is measurable JS
```
