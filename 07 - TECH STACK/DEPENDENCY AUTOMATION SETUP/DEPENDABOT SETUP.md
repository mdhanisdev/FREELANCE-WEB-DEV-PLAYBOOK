# DEPENDABOT SETUP [ DEPENDENCY AUTOMATION ]
------------------------------------------------------------------------
Dependabot is the alternative when you want GitHub-native dependency
updates with zero external app: version updates and security alerts
configured entirely in-repo. This guide sets it up for a Next.js +
TypeScript + pnpm repository.
------------------------------------------------------------------------
## STEP 1 : Enable in Repository Settings

In GitHub: Settings → Code security → enable "Dependabot alerts" and
"Dependabot security updates". These require no config file and open PRs
for vulnerable dependencies automatically.
------------------------------------------------------------------------
## STEP 2 : Add the Config File

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm" # covers pnpm workspaces too
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "05:00"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
    commit-message:
      prefix: "chore(deps)"
```

Dependabot's "npm" ecosystem understands `pnpm-lock.yaml`.

```text
Two tracks
  security alerts ─▶ urgent PRs (any time)
  version updates ─▶ scheduled PRs (weekly, Monday 05:00)
```
------------------------------------------------------------------------
## STEP 3 : Group Updates to Reduce Noise

```yaml
# .github/dependabot.yml (under the npm update entry)
    groups:
      dev-dependencies:
        dependency-type: "development"
        update-types: ["minor", "patch"]
      react-ecosystem:
        patterns: ["react", "react-dom", "next"]
```
------------------------------------------------------------------------
## STEP 4 : Auto-Merge Safe Updates

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot auto-merge
on: pull_request
permissions:
  contents: write
  pull-requests: write
jobs:
  merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: meta
      - if: steps.meta.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
------------------------------------------------------------------------
## STEP 5 : Ignore Noisy Packages (Optional)

```yaml
    ignore:
      - dependency-name: "@types/node"
        update-types: ["version-update:semver-major"]
```
------------------------------------------------------------------------
## STEP 6 : Verify

Push the config to the default branch, then open Insights → Dependency
graph → Dependabot to see the last run and any errors. Windows note:
config lives in the repo, so local OS is irrelevant; ensure the YAML uses
spaces (never tabs) — Windows editors sometimes insert tabs.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Enable security updates first — they are the highest value
✓ Use the npm ecosystem entry; it reads pnpm-lock.yaml
✓ Group dev + framework packages to cut PR volume
✓ Auto-merge only semver-patch after CI passes
✓ Gate major updates behind manual review
✓ Schedule version updates off-hours, weekly
✓ Keep YAML tab-free; validate before pushing
✓ Prefix commits (chore(deps)) for clean changelogs
```
