# BUILD & DEV WORKFLOW [ SHIP SMALL, SHIP OFTEN ]
------------------------------------------------------------------------

## WHERE YOU ARE

You picked the stack in **07 - TECH STACK/**. This guide turns that stack
into a working build loop: a clean repo, quality gates that run on every
commit, a branching model, and a weekly rhythm the client can watch.

```text
07 (choose tools) ──▶ 08 (set up the loop) ──▶ 09 (test) ──▶ 10 (launch)
```

------------------------------------------------------------------------

## REPOSITORY SETUP

```text
□ Create a PRIVATE git repository
□ Best practice: create it in the CLIENT's GitHub org from day one
  (matches the ownership rule from 06 — nothing to migrate later)
□ If building in your own org first, plan the transfer per contract
□ Add a README, .gitignore (node), and a LICENSE if the contract needs
□ Protect main: no direct pushes, require the checks below to pass
```

Client org from day one means the code, issues, and history already live
where they belong. You are invited as a collaborator and simply leave at
handoff.

------------------------------------------------------------------------

## QUALITY GATES (FROM 07)

The setup for these lives in **07 - TECH STACK/**. Wire them in now so
they run automatically — the goal is that bad code physically cannot get
committed.

```text
NEED                      SET UP IN
------------------------  ------------------------------------------
ESLint + Prettier         07 - TECH STACK/CODE QUALITY SETUP/
Git hooks (Husky)         07 - TECH STACK/GIT HOOKS SETUP/
Env variable validation   07 - TECH STACK/ENV VALIDATION SETUP/
```

```text
✓ Husky pre-commit  → lint-staged runs ESLint + Prettier on staged files
✓ Husky commit-msg  → commitlint enforces conventional commits
✓ Husky pre-push    → typecheck / build so main never breaks
✓ Env validation    → app refuses to boot with missing/invalid env vars
```

------------------------------------------------------------------------

## CONVENTIONAL COMMITS

Every commit message follows one shape. This keeps history readable and
enables automated changelogs later.

```text
<type>(optional scope): <short description>

feat:     a new feature
fix:      a bug fix
chore:    tooling, deps, config
docs:     documentation only
refactor: code change that isn't a feature or fix
test:     adding or fixing tests
style:    formatting, no logic change

examples:
  feat(auth): add magic-link sign in
  fix(cart): prevent negative quantities
  chore: bump next to 15.1
```

commitlint (installed via the GIT HOOKS SETUP) rejects anything that does
not match, so the rule enforces itself.

------------------------------------------------------------------------

## BRANCHING MODEL

Keep it simple. `main` is always deployable; real work happens on short
feature branches.

```text
main  ●─────●─────●──────────●─────●   (always green, always deployable)
       \         /            \   /
        ●───●───●              ●─●      feature/contact-form
        feature/hero-section
```

```text
✓ main         → protected, deployable at all times
✓ feature/*    → one branch per feature or fix, short-lived
✓ Open a PR into main; checks must pass before merge
✓ Delete the branch after merge; keep history tidy
```

------------------------------------------------------------------------

## CLIENT REVIEW: WEEKLY PREVIEW DEPLOYS

Vercel builds a **preview deployment** for every branch and pull request.
Use this to keep the client in the loop without giving them your laptop.

```text
□ Push a feature branch → Vercel posts a unique preview URL
□ Once a week, share the latest preview link with the client
□ Collect feedback against that specific URL (not vague "the site")
□ Turn feedback into issues; fix on branches; new preview each push
```

The client sees steady visible progress every week, and you get feedback
early instead of a giant surprise at the end.

------------------------------------------------------------------------

## SCOPE CREEP = PAID CHANGE REQUESTS

New ideas will arrive mid-build. That is normal — just never let them
quietly expand the contract for free.

```text
✓ Anything outside the agreed scope is logged as a CHANGE REQUEST
✓ Each change request gets a short estimate and a price
✓ Client approves in writing BEFORE you build it
✓ Track them in your PM tool so nothing is "just a quick thing"
```

```text
CHANGE REQUEST LOG (example)
------------------------------------------------------------------
DATE        REQUEST                    EST     PRICE    STATUS
2026-07-03  Add blog / CMS section     2 days  $X       Approved
2026-07-08  Multi-language support     4 days  $X       Pending
------------------------------------------------------------------
```

This protects your time and keeps the relationship honest.

------------------------------------------------------------------------

## DEV RHYTHM

```text
✓ Small commits — one logical change each, easy to review and revert
✓ Ship often — merge finished features to main continuously
✓ Never leave main broken — the pre-push gate is your safety net
✓ Write the test alongside the feature (see 09), not "later"
✓ Keep the client's preview URL current so progress is always visible
```

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

- **VS Code** or **Cursor** — editor with ESLint/Prettier integration.
- **GitHub** — private repo, ideally in the client's org from day one.
- **Vercel previews** — automatic per-branch review URLs for the client.
- **pnpm** — fast, disk-efficient package manager for the project.
- **07 - TECH STACK setup files** — CODE QUALITY SETUP, GIT HOOKS SETUP,
  and ENV VALIDATION SETUP wire up the gates referenced above.

------------------------------------------------------------------------

## NEXT STEP

The build loop is running and features are landing on main. Before you
hand anything over, prove it works.

➡  **09 - TESTING & QA/**  — unit, e2e, accessibility, performance, and a
   full QA checklist before launch.
