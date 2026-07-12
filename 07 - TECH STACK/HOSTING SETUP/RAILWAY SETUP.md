# RAILWAY SETUP [ HOSTING / FULL-STACK NEXT.JS + DB ]

------------------------------------------------------------------------

Railway is the simplest way to run an app and its database together.
One project holds both services, private networking links them, and
Nixpacks builds Next.js with no Dockerfile required. Great when you
want a real backend without managing infrastructure.

------------------------------------------------------------------------

## STEP 1 : Create The Project

```bash
pnpm dlx @railway/cli login
pnpm dlx @railway/cli init      # creates a new project
```

Or use the dashboard: railway.app > New Project > Deploy from GitHub
repo, then authorize and select the repository.

------------------------------------------------------------------------

## STEP 2 : Add The App Service

Railway auto-detects Next.js via Nixpacks. Set the package manager and
commands explicitly for deterministic builds.

```bash
# Service > Settings
Install:  pnpm install --frozen-lockfile
Build:    pnpm build
Start:    pnpm start
```

Bind the server to Railway's injected port.

```json
// package.json
{
  "scripts": {
    "start": "next start -p ${PORT:-3000}"
  }
}
```

------------------------------------------------------------------------

## STEP 3 : Add A Database

Click New > Database > PostgreSQL (or MySQL / Redis). Railway
provisions it and exposes a `DATABASE_URL` you reference from the app.

```text
   ┌─────────────────── Railway Project ───────────────────┐
   │                                                        │
   │   [ web service ]  ──private network──>  [ Postgres ] │
   │    Next.js app          DATABASE_URL       volume-backed│
   │        │                                               │
   └────────┼───────────────────────────────────────────────┘
            │ public domain (TLS)
          Internet
```

------------------------------------------------------------------------

## STEP 4 : Environment Variables

Set under Service > Variables. Reference other services with Railway's
variable interpolation instead of copying secrets.

```text
KEY                     VALUE
--------------------------------------------------
DATABASE_URL            ${{ Postgres.DATABASE_URL }}
NEXT_PUBLIC_API_URL     https://api.example.com
AUTH_SECRET             <generated-secret>
NODE_ENV                production
```

Browser-exposed values still require the `NEXT_PUBLIC_` prefix.

------------------------------------------------------------------------

## STEP 5 : Domain + Deploy

Generate a `*.up.railway.app` domain instantly, or add a custom domain
under Service > Settings > Networking and set the given CNAME at your
registrar.

```text
RECORD    NAME    VALUE
----------------------------------------------
CNAME     www     <service>.up.railway.app
```

```bash
pnpm dlx @railway/cli up        # deploy current dir
pnpm dlx @railway/cli logs      # tail logs
```

Every push to the connected branch triggers a new deploy; previous
deploys stay available for rollback.

------------------------------------------------------------------------

## WHEN TO CHOOSE RAILWAY

```text
✓ You need app + database + cache in one place, fast
✓ Want private networking between services out of the box
✓ Prefer usage-based pricing over managing servers
✓ Running migrations, workers, or cron alongside the web app
✗ Pure static/edge frontend — Vercel/Cloudflare fit better
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Reference secrets via ${{ Service.VAR }}, never hardcode
✓ Bind next start to $PORT so Railway routing works
✓ Use pnpm install --frozen-lockfile for reproducible builds
✓ Keep DB in the same project for private-network latency
✓ Run migrations as a deploy or pre-start step, not manually
✓ Attach a volume to stateful services to persist data
✓ Use separate environments (prod/staging) inside the project
✓ Watch usage metrics; set spend limits to avoid surprises
```
