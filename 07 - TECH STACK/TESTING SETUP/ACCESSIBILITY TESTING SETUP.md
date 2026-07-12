# ACCESSIBILITY TESTING SETUP [ A11Y TESTING ]

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add -D eslint-plugin-jsx-a11y
pnpm add -D @axe-core/playwright
```

------------------------------------------------------------------------

## STEP 2 : Configure ESLint

```js
// eslint.config.mjs
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  {
    plugins: {
      "jsx-a11y": jsxA11y,
    },
    rules: {
      ...jsxA11y.configs.recommended.rules,
    },
  },
];
```

------------------------------------------------------------------------

## STEP 3 : Install Playwright (If Not Installed)

```bash
pnpm add -D @playwright/test
pnpm exec playwright install
```

------------------------------------------------------------------------

## STEP 4 : Create Accessibility Test

`tests/accessibility.spec.ts`

```ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("homepage has no accessibility violations", async ({ page }) => {
  await page.goto("http://localhost:3000");

  const results = await new AxeBuilder({ page }).analyze();

  expect(results.violations).toEqual([]);
});
```

------------------------------------------------------------------------

## STEP 5 : Add Script

```json
{
  "scripts": {
    "a11y": "playwright test tests/accessibility.spec.ts"
  }
}
```

------------------------------------------------------------------------

## STEP 6 : Run Accessibility Test

```bash
pnpm dev
```

In another terminal:

```bash
pnpm a11y
```

------------------------------------------------------------------------

## STEP 7 : Common Accessibility Checklist

```text
✓ Every input has a label
✓ Images have alt text
✓ Buttons have accessible names
✓ Keyboard navigation works
✓ Sufficient color contrast
✓ Correct heading hierarchy
✓ ARIA attributes where needed
✓ Visible focus indicators
```

------------------------------------------------------------------------

## STEP 8 : Example

Bad

```tsx
<input />
```

Good

```tsx
<label htmlFor="email">Email</label>

<input
  id="email"
  type="email"
/>
```

------------------------------------------------------------------------

## FINAL PROJECT STRUCTURE

```text
my-app/
│
├── src/
├── tests/
│   └── accessibility.spec.ts
│
├── playwright.config.ts
├── eslint.config.mjs
└── package.json
```

------------------------------------------------------------------------

## RECOMMENDED WORKFLOW

```text
ESLint (jsx-a11y)
      │
      ▼
Playwright + axe-core
      │
      ▼
Fix Violations
      │
      ▼
Deploy
```
