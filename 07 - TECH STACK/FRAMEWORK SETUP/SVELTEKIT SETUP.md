# SVELTEKIT SETUP [ COMPILED FULL-STACK FRAMEWORK ]
------------------------------------------------------------------------
## STEP 1 : Scaffold The Project

SvelteKit uses the `sv` CLI and Vite. Svelte 5 (runes) is the default.

```bash
pnpm dlx sv create my-app
cd my-app
pnpm install
pnpm dev
```

Select "SvelteKit minimal", TypeScript syntax, and add `tailwindcss` +
`eslint` when prompted. Windows note: run the wizard in Windows Terminal
or Git Bash so arrow-key selection works.
------------------------------------------------------------------------
## STEP 2 : File-Based Routing

Routes live in `src/routes`; folders map to URL segments.

```text
src/routes/
├── +layout.svelte          # shared shell (nav, <slot/>)
├── +page.svelte            # "/"
├── about/+page.svelte      # "/about"
├── blog/
│   ├── +page.svelte        # "/blog"
│   └── [slug]/
│       ├── +page.svelte    # "/blog/:slug"
│       └── +page.ts        # load() for this route
└── (app)/dashboard/+page.svelte   # (app) is a pathless group
```

`+page.svelte` = UI, `+page.ts` = universal load, `+page.server.ts` =
server-only load/actions, `+layout.*` = shared wrappers.
------------------------------------------------------------------------
## STEP 3 : Load Functions — Fetch Data

Return data to the page. Use `+page.ts` for universal, `+page.server.ts`
for server-only (DB, secrets).

```ts
// src/routes/blog/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ params }) => {
  const post = await db.post.findUnique({ where: { slug: params.slug } });
  if (!post) throw error(404, "Not found");
  return { post };
};
```

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
  let { data } = $props();          // Svelte 5 runes
</script>
<h1>{data.post.title}</h1>
```
------------------------------------------------------------------------
## STEP 4 : Form Actions — Mutations

Named actions handle POST submissions with progressive enhancement.

```ts
// src/routes/contact/+page.server.ts
import { fail, redirect } from "@sveltejs/kit";
import type { Actions } from "./$types";

export const actions: Actions = {
  default: async ({ request }) => {
    const form = await request.formData();
    const email = String(form.get("email")); // e.g. user@example.com
    if (!email.includes("@")) return fail(400, { email, invalid: true });
    await db.lead.create({ data: { email } });
    throw redirect(303, "/thanks");
  },
};
```

```svelte
<script lang="ts">
  import { enhance } from "$app/forms";
</script>
<form method="POST" use:enhance>
  <input name="email" type="email" required />
  <button>Subscribe</button>
</form>
```

`use:enhance` upgrades the native form into a client transition without
losing no-JS functionality.
------------------------------------------------------------------------
## STEP 5 : Request Flow Overview

```text
Request / <form> POST
   │
   ▼
+page.server.ts actions{}  ──►  fail() | redirect() | data
   │
   ▼
load() (server/universal) re-runs and returns data
   │
   ▼
+layout.svelte ► +page.svelte  render with $props().data
```
------------------------------------------------------------------------
## STEP 6 : Choose An Adapter, Then Build

`adapter-auto` detects common hosts; pin an explicit adapter for control.

```bash
pnpm add -D @sveltejs/adapter-node
```

```ts
// svelte.config.js
import adapter from "@sveltejs/adapter-node";
export default { kit: { adapter: adapter() } };
```

```bash
pnpm build
pnpm preview
```
------------------------------------------------------------------------
## STEP 7 : When To Use SvelteKit

Choose SvelteKit when you want the smallest client bundles and the
simplest reactivity model — the compiler ships little runtime, and runes
(`$state`, `$props`, `$derived`) replace hooks/boilerplate. Great for
dashboards, interactive apps, and content sites alike. Prefer React-based
frameworks (Next.js / Remix) when your team, hiring, or component
ecosystem is React-centric.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Put secrets and DB access in +page.server.ts, never +page.ts
✓ Use form actions + use:enhance for progressive-enhancement writes
✓ Return fail(status, data) for validation instead of throwing
✓ Rely on generated ./$types for typed load and actions
✓ Organize with pathless (group) folders and shared +layout files
✓ Pin an explicit adapter for production instead of adapter-auto
✓ Prefer Svelte 5 runes ($state/$props) over legacy stores where apt
✓ Keep universal load pure; do side effects on the server
```
