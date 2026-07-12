# CLOUDFLARE PAGES SETUP [ HOSTING / EDGE NEXT.JS ]

------------------------------------------------------------------------

Deploy Next.js to Cloudflare's global edge network. Runs on Workers via
an adapter — either `@cloudflare/next-on-pages` (Pages) or OpenNext for
Cloudflare (Workers). Cheapest at scale, lowest global latency.

------------------------------------------------------------------------

## STEP 1 : Choose An Adapter

```text
   @cloudflare/next-on-pages   ──> Cloudflare Pages, edge runtime
                                   mature, App Router support

   @opennextjs/cloudflare      ──> Cloudflare Workers, node compat
                                   fuller Next.js feature coverage
```

Use OpenNext when you need Node APIs / ISR; use next-on-pages for a
pure edge deployment.

------------------------------------------------------------------------

## STEP 2 : Install (next-on-pages path)

```bash
pnpm add -D @cloudflare/next-on-pages wrangler
```

Set edge runtime on dynamic routes that use server code.

```text
// app/api/hello/route.ts
export const runtime = "edge";
```

------------------------------------------------------------------------

## STEP 3 : Configure wrangler.toml

```yaml
name = "my-next-app"
compatibility_date = "2026-07-01"
compatibility_flags = ["nodejs_compat"]
pages_build_output_dir = ".vercel/output/static"
```

Add a build script to package.json.

```json
{
  "scripts": {
    "pages:build": "pnpm dlx @cloudflare/next-on-pages",
    "pages:deploy": "wrangler pages deploy .vercel/output/static"
  }
}
```

------------------------------------------------------------------------

## STEP 4 : Connect + Deploy

- Dashboard: Workers & Pages > Create > Pages > Connect to Git.
- Build command: `pnpm pages:build`.
- Output directory: `.vercel/output/static`.

```bash
# or deploy from CLI
pnpm pages:build
pnpm wrangler pages deploy .vercel/output/static
```

------------------------------------------------------------------------

## STEP 5 : Environment Variables + Bindings

Set vars under the Pages project > Settings > Environment variables
(Production and Preview). Native Cloudflare resources attach as
bindings rather than URLs.

```text
KEY / BINDING           TYPE          SCOPE
------------------------------------------------
NEXT_PUBLIC_API_URL     plaintext     prod + preview
AUTH_SECRET             secret        prod + preview
MY_KV                   KV namespace  binding
MY_DB                   D1 database   binding
```

------------------------------------------------------------------------

## STEP 6 : Custom Domain

Add the domain under the Pages project > Custom domains. If DNS is
already on Cloudflare, records and TLS are configured automatically.

```text
   git push branch ──> PREVIEW deploy (*.pages.dev)
   push to main ─────> PRODUCTION deploy (custom domain, edge)
```

------------------------------------------------------------------------

## WHEN TO CHOOSE CLOUDFLARE

```text
✓ Global audience — run at the edge, near every user
✓ Cost-sensitive at high traffic (generous free/cheap tiers)
✓ Want native KV / D1 / R2 / Durable Objects bindings
✗ App relies on long-running Node servers or heavy ISR
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Pick the adapter first: next-on-pages (edge) vs OpenNext (Workers)
✓ Set export const runtime = "edge" on server routes for Pages
✓ Enable nodejs_compat only when a dependency needs it
✓ Use bindings (KV/D1/R2) instead of external service URLs
✓ Keep compatibility_date current and pinned in wrangler.toml
✓ Scope secrets per environment via the dashboard, not code
✓ Test the edge build locally with wrangler pages dev
✓ Prefer Cloudflare DNS for automatic managed TLS
```
