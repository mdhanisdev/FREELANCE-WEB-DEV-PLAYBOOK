# TURBOREPO SETUP [ ALTERNATIVE STACKS ]

------------------------------------------------------------------------

Turborepo is a high-performance build system for JavaScript monorepos. It
caches task outputs, runs work in parallel across packages, and only
rebuilds what changed. Use it to share code between your Next.js web app,
mobile app, and internal libraries in one repo.

------------------------------------------------------------------------

## STEP 1 : Scaffold The Monorepo

```bash
pnpm dlx create-turbo@latest my-repo
cd my-repo
```

This creates a pnpm workspace with a sample `web` app and shared `ui`
package already linked.

------------------------------------------------------------------------

## STEP 2 : Understand The Layout

```text
   my-repo/
   ├── apps/
   │   ├── web/          → Next.js App Router
   │   └── docs/         → second Next.js app
   ├── packages/
   │   ├── ui/           → shared React components
   │   ├── config/       → eslint + tsconfig presets
   │   └── db/           → shared Drizzle schema/client
   ├── turbo.json        → task pipeline
   └── pnpm-workspace.yaml
```

------------------------------------------------------------------------

## STEP 3 : Declare Workspaces

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

```json
// apps/web/package.json — consume a shared package
{
  "dependencies": {
    "@repo/ui": "workspace:*",
    "@repo/db": "workspace:*"
  }
}
```

The `workspace:*` protocol links packages locally without publishing.

------------------------------------------------------------------------

## STEP 4 : Configure The Task Pipeline

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "lint": {},
    "dev": { "cache": false, "persistent": true }
  }
}
```

`^build` means "build this package's dependencies first."

------------------------------------------------------------------------

## STEP 5 : Run Tasks With Caching

```bash
pnpm turbo build          # builds everything, caches outputs
pnpm turbo build          # second run: FULL TURBO (instant cache hit)
pnpm turbo dev            # runs all dev servers in parallel
pnpm turbo lint --filter=web  # scope to one package
```

```text
   turbo build
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │ packages │──▶│ apps/web │   │ apps/docs│
   │   /db    │   └──────────┘   └──────────┘
   └──────────┘         ▲
        └── ^build ordering ──┘
   Unchanged packages restore from cache instead of rebuilding.
```

------------------------------------------------------------------------

## STEP 6 : Remote Cache In CI

```bash
pnpm dlx turbo login
pnpm dlx turbo link      # connect to Vercel Remote Cache
```

CI shares the cache across machines, so a build already done locally or
on another runner is restored instead of repeated. On Windows, run turbo
from Git Bash or PowerShell; paths in `outputs` use forward slashes.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Put shared code in packages/ and link with workspace:*
✓ Declare outputs in turbo.json so caching works correctly
✓ Use ^build dependsOn to enforce dependency build order
✓ Mark dev tasks cache:false and persistent:true
✓ Scope runs with --filter to touch only what you need
✓ Enable remote cache to share builds across CI and teammates
✓ Share one tsconfig/eslint preset via a config package
✓ Never cache tasks with side effects or secrets
✓ Keep app-specific code in apps/, reusable code in packages/
```
