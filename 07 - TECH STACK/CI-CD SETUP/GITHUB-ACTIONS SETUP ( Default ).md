# GITHUB ACTIONS SETUP [ CI/CD / NEXT.JS PIPELINE ]

------------------------------------------------------------------------

GitHub Actions runs your quality gate on every push and PR: install,
lint, typecheck, test, build — plus an optional end-to-end job. Wire it
to branch protection so nothing merges to main without a green run.

------------------------------------------------------------------------

## STEP 1 : Pipeline Overview

```text
   push / pull_request
          │
          ▼
   ┌───────────────────────────────────────────┐
   │  ci  (ubuntu-latest, node 20, pnpm 9)      │
   │  install ─> lint ─> typecheck ─> test ─> build
   └───────────────────────────────────────────┘
          │ (main only, after ci passes)
          ▼
   ┌───────────────────────────────────────────┐
   │  e2e (optional)  playwright against build  │
   └───────────────────────────────────────────┘
```

------------------------------------------------------------------------

## STEP 2 : Create ci.yml

Commit to `.github/workflows/ci.yml`.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Typecheck
        run: pnpm exec tsc --noEmit

      - name: Test
        run: pnpm test

      - name: Build
        run: pnpm build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.NEXT_PUBLIC_API_URL }}
```

------------------------------------------------------------------------

## STEP 3 : Package Scripts

Make sure these exist so the workflow steps resolve.

```json
{
  "scripts": {
    "lint": "next lint",
    "test": "vitest run",
    "build": "next build"
  }
}
```

------------------------------------------------------------------------

## STEP 4 : Secrets + Variables

- Repo > Settings > Secrets and variables > Actions.
- Secrets (masked): `DATABASE_URL`, `AUTH_SECRET`.
- Variables (plain): `NEXT_PUBLIC_API_URL`.

```yaml
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      AUTH_SECRET: ${{ secrets.AUTH_SECRET }}
```

Never echo secrets to logs; Actions masks them but avoid printing.

------------------------------------------------------------------------

## STEP 5 : Optional E2E Job

Runs only after `ci` succeeds. Uses the official Playwright image.

```yaml
  e2e:
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    container: mcr.microsoft.com/playwright:v1.48.0-jammy
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - run: pnpm exec playwright test
```

------------------------------------------------------------------------

## STEP 6 : Branch Protection

Settings > Branches > Add rule for `main`:

```text
✓ Require a pull request before merging
✓ Require status checks to pass  ──> select "ci"
✓ Require branches to be up to date before merging
✓ Do not allow bypassing the above settings
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always install with --frozen-lockfile in CI
✓ Cache pnpm via setup-node cache: pnpm to speed runs
✓ Cancel in-progress runs per ref with a concurrency group
✓ Keep secrets in Actions Secrets; use Variables for public config
✓ Pin action versions (@v4) for reproducibility
✓ Gate merges to main on the ci status check
✓ Run e2e only after unit gate passes to save minutes
✓ Fail fast: lint and typecheck before the slower build
```
