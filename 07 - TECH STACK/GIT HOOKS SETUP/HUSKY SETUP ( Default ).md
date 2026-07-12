# HUSKY SETUP [ GIT HOOKS / COMMIT GUARDRAILS ]
------------------------------------------------------------------------
Husky manages native Git hooks so quality gates run automatically on
commit and push. This reference wires pre-commit, pre-push, and a
commit-msg hook backed by commitlint for Conventional Commits.
------------------------------------------------------------------------
## STEP 1 : Install Husky

```bash
pnpm add -D husky
```

Husky v9+ stores hooks in a version-controlled `.husky/` directory,
so every teammate gets identical hooks after `pnpm install`.
------------------------------------------------------------------------
## STEP 2 : Initialize Husky

```bash
pnpm exec husky init
```

This creates `.husky/pre-commit` and adds a `prepare` script to
`package.json` so hooks auto-install on clone:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```
------------------------------------------------------------------------
## STEP 3 : Configure pre-commit

Edit `.husky/pre-commit` to run quality gates before each commit:

```sh
pnpm lint
pnpm typecheck
pnpm test --run
```

Add the matching scripts to `package.json`:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "test": "vitest"
  }
}
```

For faster commits, delegate to lint-staged (see LINT-STAGED SETUP).
------------------------------------------------------------------------
## STEP 4 : Configure pre-push

Create `.husky/pre-push` for heavier checks that should not block
every commit but must pass before code leaves the machine:

```sh
pnpm typecheck
pnpm build
```

```bash
chmod +x .husky/pre-push
```

On Windows, Git Bash honors the shebang; no extra config needed.
------------------------------------------------------------------------
## STEP 5 : Configure commit-msg with commitlint

```bash
pnpm add -D @commitlint/cli @commitlint/config-conventional
```

Create `commitlint.config.mjs`:

```js
export default { extends: ["@commitlint/config-conventional"] };
```

Create `.husky/commit-msg`:

```sh
pnpm exec commitlint --edit "$1"
```

```text
   git commit
       |
       v
  pre-commit  --> lint + typecheck + test
       |
       v
  commit-msg  --> commitlint (feat|fix|chore: ...)
       |
       v   git push
  pre-push    --> typecheck + build
       |
       v
   remote accepts commit
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep the prepare: "husky" script so hooks install on clone
✓ Commit the .husky/ directory to version control
✓ Put fast checks in pre-commit, slow ones in pre-push
✓ Delegate pre-commit linting to lint-staged for speed
✓ Enforce Conventional Commits via commit-msg + commitlint
✓ Never use --no-verify to bypass hooks on shared branches
✓ Use pnpm exec so hooks resolve local binaries reliably
✓ Test hooks in Git Bash on Windows to confirm shebangs work
```
