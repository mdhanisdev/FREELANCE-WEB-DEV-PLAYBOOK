# ESLINT SETUP [ CODE QUALITY / STATIC ANALYSIS ]
------------------------------------------------------------------------
ESLint enforces consistent, bug-resistant TypeScript across a Next.js
App Router project. This reference uses the modern flat config
(`eslint.config.mjs`) with the official Next.js presets.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add -D eslint @eslint/eslintrc eslint-config-next typescript
```

`eslint-config-next` ships both `next/core-web-vitals` and
`next/typescript`. The `@eslint/eslintrc` package provides the
`FlatCompat` bridge needed to consume legacy shareable configs.
------------------------------------------------------------------------
## STEP 2 : Create the Flat Config

Create `eslint.config.mjs` at the project root:

```js
import { dirname } from "path";
import { fileURLToPath } from "url";
import { FlatCompat } from "@eslint/eslintrc";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const compat = new FlatCompat({ baseDirectory: __dirname });

const eslintConfig = [
  ...compat.extends("next/core-web-vitals", "next/typescript"),
  {
    rules: {
      "no-console": ["warn", { allow: ["warn", "error"] }],
      "prefer-const": "error",
      "eqeqeq": ["error", "always"],
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
      "@typescript-eslint/consistent-type-imports": "error",
    },
  },
];

export default eslintConfig;
```
------------------------------------------------------------------------
## STEP 3 : Configure Ignores

Ignore patterns live inside the flat config as a dedicated object.
Never rely on the deprecated `.eslintignore` file with flat config.

```js
{
  ignores: [
    ".next/**",
    "node_modules/**",
    "dist/**",
    "build/**",
    "coverage/**",
    "next-env.d.ts",
    "*.config.mjs",
  ],
}
```

Append this object to the exported array before `export default`.
------------------------------------------------------------------------
## STEP 4 : Add Package Scripts

Wire linting into `package.json`:

```json
{
  "scripts": {
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "lint:strict": "next lint --max-warnings 0"
  }
}
```

Use `lint:strict` in CI so warnings become blocking failures.
------------------------------------------------------------------------
## STEP 5 : Verify the Setup

```bash
pnpm lint
pnpm lint:fix
```

```text
   +-----------------------+
   |   eslint.config.mjs   |
   +-----------+-----------+
               |
        FlatCompat.extends
               |
   +-----------v-----------+
   | next/core-web-vitals  |
   | next/typescript       |
   +-----------+-----------+
               |
       custom rule layer
               |
   +-----------v-----------+
   |  app/ src/ *.ts *.tsx |
   +-----------------------+
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use flat config (eslint.config.mjs), not legacy .eslintrc
✓ Extend next/core-web-vitals AND next/typescript together
✓ Declare ignores inside the config, never .eslintignore
✓ Run lint:strict with --max-warnings 0 in CI pipelines
✓ Prefer consistent-type-imports for smaller bundles
✓ Allow only warn/error for no-console in production code
✓ Let Prettier own formatting; ESLint owns correctness
✓ Keep the config committed so every teammate shares rules
```
