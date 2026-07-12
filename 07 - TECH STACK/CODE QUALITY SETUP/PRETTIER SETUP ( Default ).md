# PRETTIER SETUP [ CODE QUALITY / FORMATTING ]
------------------------------------------------------------------------
Prettier is the single source of truth for code formatting. Paired with
`prettier-plugin-tailwindcss`, it also sorts Tailwind classes into the
canonical order so diffs stay clean across the whole team.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add -D prettier prettier-plugin-tailwindcss
```

The Tailwind plugin must load last so it can run after any other
Prettier plugins that transform the same nodes.
------------------------------------------------------------------------
## STEP 2 : Create prettier.config.mjs

Create `prettier.config.mjs` at the project root:

```js
/** @type {import("prettier").Config} */
const config = {
  semi: true,
  singleQuote: false,
  trailingComma: "all",
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  arrowParens: "always",
  endOfLine: "lf",
  plugins: ["prettier-plugin-tailwindcss"],
  tailwindFunctions: ["clsx", "cn", "cva"],
};

export default config;
```

`tailwindFunctions` tells the plugin which helper calls also contain
class strings to sort (common with `clsx`, `cn`, and `cva`).
------------------------------------------------------------------------
## STEP 3 : Create .prettierignore

```text
.next/
node_modules/
dist/
build/
coverage/
pnpm-lock.yaml
*.min.js
next-env.d.ts
```

Never format the lockfile or generated build output.
------------------------------------------------------------------------
## STEP 4 : Add Format Scripts

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

Use `format:check` in CI; it fails when any file is not formatted.
------------------------------------------------------------------------
## STEP 5 : VS Code Format-On-Save

Create `.vscode/settings.json` so Windows teammates share behavior:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "files.eol": "\n",
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

```text
   save file
      |
      v
  Prettier (format)  --->  prettier-plugin-tailwindcss (sort classes)
      |
      v
  ESLint --fix (source.fixAll.eslint)
      |
      v
   clean, sorted, valid file on disk
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep prettier-plugin-tailwindcss last in the plugins array
✓ Set endOfLine: "lf" so Windows CRLF never leaks into diffs
✓ Run format:check in CI, format:write locally
✓ List clsx/cn/cva in tailwindFunctions for full class sorting
✓ Never lint formatting rules in ESLint; let Prettier own them
✓ Commit .vscode/settings.json for consistent format-on-save
✓ Add pnpm-lock.yaml and build output to .prettierignore
✓ Use one config file (prettier.config.mjs) checked into git
```
