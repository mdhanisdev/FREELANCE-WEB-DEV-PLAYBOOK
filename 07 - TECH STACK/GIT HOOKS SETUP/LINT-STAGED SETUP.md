# LINT-STAGED SETUP [ GIT HOOKS / FAST COMMITS ]
------------------------------------------------------------------------
lint-staged runs linters and formatters only against files staged for
commit, instead of the whole repository. This keeps pre-commit hooks
fast while still guaranteeing every committed file is clean.
------------------------------------------------------------------------
## STEP 1 : Install lint-staged

```bash
pnpm add -D lint-staged
```

This assumes Husky, ESLint, and Prettier are already configured
(see HUSKY SETUP, ESLINT SETUP, and PRETTIER SETUP).
------------------------------------------------------------------------
## STEP 2 : Create the Config

Create `lint-staged.config.mjs` at the project root:

```js
/** @type {import("lint-staged").Configuration} */
const config = {
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{js,jsx,mjs,cjs}": ["eslint --fix", "prettier --write"],
  "*.{json,md,css,yml,yaml}": ["prettier --write"],
};

export default config;
```

Commands run left to right; `eslint --fix` first, then
`prettier --write` formats the already-fixed output.
------------------------------------------------------------------------
## STEP 3 : Wire Into Husky pre-commit

Replace the body of `.husky/pre-commit` so it calls lint-staged
instead of scanning the entire project:

```sh
pnpm exec lint-staged
```

lint-staged re-stages the fixed files automatically, so the
formatted result is what actually gets committed.
------------------------------------------------------------------------
## STEP 4 : Keep Type Checking Whole-Project

`tsc` cannot type-check isolated files reliably, so leave it out of
lint-staged and run it separately in pre-commit or CI:

```sh
pnpm exec lint-staged
pnpm typecheck
```

Type errors often live in files you did not touch this commit, so a
project-wide `tsc --noEmit` is the correct scope.
------------------------------------------------------------------------
## STEP 5 : Verify

```bash
git add src/example.tsx
git commit -m "feat: add example"
```

```text
   git commit
       |
       v
  .husky/pre-commit
       |
       v
  pnpm exec lint-staged
       |
   +---+--------------------------+
   | staged *.ts,*.tsx only       |
   |   eslint --fix               |
   |   prettier --write           |
   |   git add (re-stage fixed)   |
   +---+--------------------------+
       |
       v
  pnpm typecheck (whole project)
       |
       v
   commit succeeds
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Lint only staged files to keep commits fast
✓ Order commands: eslint --fix before prettier --write
✓ Let lint-staged re-stage fixed files automatically
✓ Keep tsc --noEmit whole-project, outside lint-staged
✓ Glob by extension so each file type gets the right tools
✓ Never pass filenames to tsc inside lint-staged
✓ Match ESLint/Prettier configs used by your editor
✓ Run the full suite in CI as a backstop for the hook
```
