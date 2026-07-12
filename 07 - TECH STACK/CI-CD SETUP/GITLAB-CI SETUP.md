# GITLAB CI SETUP [ CI/CD / NEXT.JS PIPELINE ]

------------------------------------------------------------------------

GitLab CI runs the same quality gate as GitHub Actions but is built
into GitLab itself: one `.gitlab-ci.yml`, staged jobs, shared caching,
and merge-request pipelines. Choose it when your repo already lives on
GitLab (self-managed or gitlab.com).

------------------------------------------------------------------------

## STEP 1 : Pipeline Overview

```text
   push / merge_request
          │
          ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │ install  │─> │ quality  │─> │  test    │─> │  build   │
   │ (cache)  │   │ lint+tsc │   │ vitest   │   │ next     │
   └──────────┘   └──────────┘   └──────────┘   └──────────┘
                                       │ (main only)
                                       ▼
                                 ┌──────────┐
                                 │   e2e    │
                                 └──────────┘
```

------------------------------------------------------------------------

## STEP 2 : Create .gitlab-ci.yml

Commit at the repo root.

```yaml
image: node:20-alpine

stages:
  - install
  - quality
  - test
  - build

variables:
  PNPM_HOME: "$CI_PROJECT_DIR/.pnpm-store"

# shared pnpm store cache keyed by lockfile
cache:
  key:
    files:
      - pnpm-lock.yaml
  paths:
    - .pnpm-store

before_script:
  - corepack enable
  - corepack prepare pnpm@9 --activate
  - pnpm config set store-dir .pnpm-store
```

------------------------------------------------------------------------

## STEP 3 : Define The Jobs

```yaml
install:
  stage: install
  script:
    - pnpm install --frozen-lockfile
  artifacts:
    paths:
      - node_modules
    expire_in: 1h

lint:
  stage: quality
  script:
    - pnpm lint

typecheck:
  stage: quality
  script:
    - pnpm exec tsc --noEmit

test:
  stage: test
  script:
    - pnpm test

build:
  stage: build
  script:
    - pnpm build
  artifacts:
    paths:
      - .next
    expire_in: 1h
```

Jobs in the same stage run in parallel; stages run in order.

------------------------------------------------------------------------

## STEP 4 : Secrets (CI/CD Variables)

Settings > CI/CD > Variables. Mark secrets Masked and Protected so they
only reach protected branches.

```text
KEY                     FLAGS
------------------------------------------------
DATABASE_URL            Masked, Protected
AUTH_SECRET             Masked, Protected
NEXT_PUBLIC_API_URL     plain (build-time public)
```

They are injected as environment variables automatically — no need to
declare them in the YAML.

------------------------------------------------------------------------

## STEP 5 : Optional E2E Job

```yaml
e2e:
  stage: build
  image: mcr.microsoft.com/playwright:v1.48.0-jammy
  needs: [build]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  script:
    - pnpm install --frozen-lockfile
    - pnpm exec playwright test
```

------------------------------------------------------------------------

## STEP 6 : Protect main

Settings > Repository > Protected branches: protect `main`, allow merge
via Merge Request only. Then Settings > Merge requests:

```text
✓ Pipelines must succeed before merge
✓ All threads must be resolved
✓ Require approval from code owners
```

------------------------------------------------------------------------

## WHEN TO CHOOSE GITLAB CI

```text
✓ Repo already hosted on GitLab (self-managed or SaaS)
✓ Want CI, registry, and runners in one integrated platform
✓ Need self-hosted runners behind a corporate firewall
✗ Repo lives on GitHub — use GitHub Actions instead
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Cache the pnpm store keyed by pnpm-lock.yaml
✓ Install once with --frozen-lockfile; pass node_modules as artifact
✓ Split lint/typecheck/test/build into ordered stages
✓ Mark secrets Masked + Protected; never inline them in YAML
✓ Pin the node and pnpm versions for reproducible pipelines
✓ Use rules: to run e2e only on main and save runner minutes
✓ Require green pipelines before merge via branch protection
✓ Expire build artifacts quickly to keep storage lean
```
