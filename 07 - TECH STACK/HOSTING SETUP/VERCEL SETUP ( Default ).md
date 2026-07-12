# VERCEL SETUP [ HOSTING / NEXT.JS APP ROUTER ]

------------------------------------------------------------------------

Vercel is the first-party host for Next.js. Zero-config builds, native
App Router + edge/serverless support, automatic preview deploys per
branch. Recommended default for most Next.js projects.

------------------------------------------------------------------------

## STEP 1 : Prepare The Repo

Push your Next.js project to GitHub, GitLab, or Bitbucket. Verify a
clean local build before importing.

```bash
pnpm install
pnpm build
git add -A
git commit -m "chore: ready for vercel"
git push origin main
```

------------------------------------------------------------------------

## STEP 2 : Import The Repo

- Go to vercel.com/new and select your Git provider.
- Pick the repository. Vercel auto-detects Next.js.
- Framework Preset: `Next.js`. Build command and output are inferred.

Set the package manager explicitly so builds are deterministic.

```bash
# Install Command (Project Settings > General)
pnpm install --frozen-lockfile

# Build Command
pnpm build
```

------------------------------------------------------------------------

## STEP 3 : Environment Variables (Prod / Preview / Dev)

Add vars under Project Settings > Environment Variables. Scope each to
the correct environment. Never commit secrets.

```text
KEY                     PRODUCTION   PREVIEW   DEVELOPMENT
--------------------------------------------------------------
DATABASE_URL              yes          yes         no
NEXT_PUBLIC_API_URL       yes          yes         yes
AUTH_SECRET               yes          yes         no
STRIPE_SECRET_KEY         yes          yes(test)   no
```

Only variables prefixed `NEXT_PUBLIC_` are exposed to the browser.

```bash
# Optional: manage via CLI
pnpm add -g vercel
vercel env add DATABASE_URL production
vercel env pull .env.local   # sync remote vars locally
```

------------------------------------------------------------------------

## STEP 4 : Custom Domain + DNS

Add the domain under Project Settings > Domains, then update DNS at
your registrar.

```text
RECORD    NAME    VALUE                    NOTES
------------------------------------------------------------
A         @       76.76.21.21              apex domain
CNAME     www     cname.vercel-dns.com     www subdomain
```

Vercel provisions and renews TLS certificates automatically once DNS
propagates. Set the primary domain to redirect `www` to apex (or
vice-versa) under the Domains panel.

------------------------------------------------------------------------

## STEP 5 : Preview vs Production Deploys

```text
   git push feature/x ──> PREVIEW DEPLOY
                          unique URL: proj-git-feature-x.vercel.app
                          uses PREVIEW env vars
                          comment posted on the PR

   merge to main ──────> PRODUCTION DEPLOY
                          serves custom domain
                          uses PRODUCTION env vars
                          previous deploy kept for instant rollback
```

- Every branch/PR gets an isolated Preview URL.
- `main` (Production Branch) publishes to your live domain.
- Roll back instantly from Deployments > (…) > Promote to Production.

------------------------------------------------------------------------

## STEP 6 : Verify

```bash
vercel --prod            # manual production deploy from CLI
vercel logs <deployment> # tail runtime logs
```

Check the Deployments tab for build logs, function regions, and
analytics.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use pnpm install --frozen-lockfile for reproducible builds
✓ Scope secrets per environment; never expose without NEXT_PUBLIC_
✓ Keep the Production Branch protected and require PR review
✓ Use Preview deploys for QA before merging to main
✓ Prefer vercel env pull over hand-copying .env files
✓ Enable automatic TLS; never hardcode certs
✓ Keep instant rollback in mind — never force-delete old deploys
✓ Set region close to your database to cut function latency
```
