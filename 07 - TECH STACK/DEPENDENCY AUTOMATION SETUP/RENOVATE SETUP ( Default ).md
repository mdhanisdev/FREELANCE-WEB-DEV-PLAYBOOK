# RENOVATE SETUP [ DEPENDENCY AUTOMATION ]
------------------------------------------------------------------------
Renovate is our default dependency-update bot: highly configurable,
grouping, auto-merge, and lockfile-aware for pnpm. This guide installs
the Renovate GitHub App and configures it for a Next.js + TypeScript +
pnpm repository.
------------------------------------------------------------------------
## STEP 1 : Install the GitHub App

Go to github.com/apps/renovate → Install → select your repository.
Renovate opens an onboarding "Configure Renovate" pull request
automatically. No local install is required for the hosted app.
------------------------------------------------------------------------
## STEP 2 : Add the Config File

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended", ":dependencyDashboard"],
  "timezone": "Europe/Paris",
  "schedule": ["before 6am on monday"],
  "packageManager": "pnpm",
  "labels": ["dependencies"],
  "prConcurrentLimit": 5
}
```

```text
Renovate cycle
  scan repo ─▶ find outdated deps ─▶ open grouped PRs ─▶ CI runs
        ▲                                                   │
        └───────── auto-merge if green + rules met ─────────┘
```
------------------------------------------------------------------------
## STEP 3 : Group & Schedule Updates

```json
// renovate.json (packageRules)
{
  "packageRules": [
    {
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "groupName": "dev dependencies",
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^next", "^react"],
      "groupName": "next + react"
    }
  ]
}
```
------------------------------------------------------------------------
## STEP 4 : Enable Lockfile Maintenance

```json
// renovate.json
{
  "lockFileMaintenance": {
    "enabled": true,
    "schedule": ["before 6am on the first day of the month"]
  }
}
```

Renovate refreshes `pnpm-lock.yaml` transitive deps on this schedule.
------------------------------------------------------------------------
## STEP 5 : Self-Host Alternative (Optional)

```yaml
# .github/workflows/renovate.yml
name: Renovate
on:
  schedule: [{ cron: "23 3 * * 1" }]
  workflow_dispatch:
jobs:
  renovate:
    runs-on: ubuntu-latest
    steps:
      - uses: renovatebot/github-action@v40
        with:
          token: ${{ secrets.RENOVATE_TOKEN }}
```
------------------------------------------------------------------------
## STEP 6 : Verify

Merge the onboarding PR, then open the "Dependency Dashboard" issue
Renovate creates. It lists every pending and rate-limited update and lets
you trigger runs. Windows note: config is repo-side, so local OS does not
matter; validate JSON with `pnpm dlx renovate-config-validator`.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Extend config:recommended instead of configuring from scratch
✓ Enable the dependency dashboard for visibility
✓ Auto-merge only patch/minor devDependencies, gate majors
✓ Group related packages (next + react) to cut PR noise
✓ Schedule runs off-hours to avoid CI contention
✓ Turn on lockFileMaintenance to refresh transitive deps
✓ Require green CI before any auto-merge
✓ Validate renovate.json with the config validator in CI
```
