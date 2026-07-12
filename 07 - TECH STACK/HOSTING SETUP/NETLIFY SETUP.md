# NETLIFY SETUP [ HOSTING / NEXT.JS APP ROUTER ]

------------------------------------------------------------------------

Netlify hosts Next.js via the official `@netlify/plugin-nextjs`. Strong
build pipeline, generous free tier, form handling and edge functions.
A good pick when your team already lives in the Netlify ecosystem.

------------------------------------------------------------------------

## STEP 1 : Prepare The Repo

```bash
pnpm install
pnpm build
git add -A
git commit -m "chore: ready for netlify"
git push origin main
```

------------------------------------------------------------------------

## STEP 2 : Configure netlify.toml

Commit a `netlify.toml` at the repo root. The Next.js runtime plugin
maps App Router routes to Netlify Functions and edge automatically.

```yaml
[build]
  command = "pnpm build"
  publish = ".next"

[build.environment]
  NPM_FLAGS = "--version"   # let pnpm handle install
  PNPM_VERSION = "9"

[[plugins]]
  package = "@netlify/plugin-nextjs"
```

------------------------------------------------------------------------

## STEP 3 : Import The Site

- Go to app.netlify.com > Add new site > Import an existing project.
- Connect Git provider and select the repository.
- Build command `pnpm build`, publish directory `.next` (from toml).

```bash
# Optional CLI flow
pnpm add -g netlify-cli
netlify login
netlify init
netlify deploy --build --prod
```

------------------------------------------------------------------------

## STEP 4 : Environment Variables

Set under Site configuration > Environment variables. Netlify supports
per-context scoping (production, deploy-preview, branch-deploy).

```text
KEY                     PRODUCTION   DEPLOY-PREVIEW
----------------------------------------------------
DATABASE_URL              yes           yes
NEXT_PUBLIC_API_URL       yes           yes
AUTH_SECRET               yes           yes
```

Browser-exposed vars must be prefixed `NEXT_PUBLIC_`.

------------------------------------------------------------------------

## STEP 5 : Custom Domain + DNS

Add the domain under Domain management, then point DNS at Netlify.

```text
RECORD    NAME    VALUE                       NOTES
---------------------------------------------------------
A         @       75.2.60.5                   Netlify load balancer
CNAME     www     <site>.netlify.app          www subdomain
```

Or delegate the whole zone to Netlify DNS for managed records and
automatic Let's Encrypt TLS.

------------------------------------------------------------------------

## STEP 6 : Deploy Contexts

```text
   PR / branch push ──> DEPLOY PREVIEW
                        deploy-preview-42--site.netlify.app
                        uses deploy-preview env context

   push to main ──────> PRODUCTION
                        serves custom domain
                        uses production env context
```

------------------------------------------------------------------------

## WHEN TO CHOOSE NETLIFY

```text
✓ Team already uses Netlify for other sites
✓ Need built-in form handling / split testing
✓ Want a strong static + SSR hybrid pipeline
✗ Bleeding-edge Next.js features — Vercel ships them first
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Commit netlify.toml so builds are reproducible
✓ Use @netlify/plugin-nextjs — do not hand-roll the adapter
✓ Scope secrets per deploy context, not globally
✓ Pin PNPM_VERSION to match local development
✓ Use Deploy Previews for every PR before merge
✓ Delegate DNS to Netlify for automatic managed TLS
✓ Keep NEXT_PUBLIC_ prefix discipline for client vars
✓ Enable rollback via Deploys > publish an earlier build
```
