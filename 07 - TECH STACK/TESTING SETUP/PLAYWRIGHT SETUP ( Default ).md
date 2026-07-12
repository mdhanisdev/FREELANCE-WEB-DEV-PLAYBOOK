# PLAYWRIGHT SETUP [ END-TO-END (E2E) TESTING ]

------------------------------------------------------------------------

## STEP 1 : Install Playwright

```bash
pnpm add -D @playwright/test
pnpm exec playwright install
```

------------------------------------------------------------------------

## STEP 2 : Initialize Playwright

```bash
pnpm exec playwright init
```

This creates:

```text
playwright.config.ts
tests/
```

------------------------------------------------------------------------

## STEP 3 : Configure /playwright.config.ts

```ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",

  use: {
    baseURL: "http://localhost:3000",
    headless: true,
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },

  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: true,
  },
});
```

------------------------------------------------------------------------

## STEP 4 : Update /package.json

```json
{
  "scripts": {
    "e2e": "playwright test",
    "e2e:ui": "playwright test --ui",
    "e2e:headed": "playwright test --headed",
    "e2e:debug": "playwright test --debug",
    "e2e:report": "playwright show-report"
  }
}
```

------------------------------------------------------------------------

## STEP 5 : Create Your First Test

`/tests/home.spec.ts`

```ts
import { test, expect } from "@playwright/test";

test("homepage loads", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveTitle(/Next/i);
});
```

------------------------------------------------------------------------

## STEP 6 : Login Flow Test

`/tests/login.spec.ts`

```ts
import { test, expect } from "@playwright/test";

test("user can login", async ({ page }) => {
  await page.goto("/login");

  await page.fill('input[name="email"]', "hanis@test.com");
  await page.fill('input[name="password"]', "123456");

  await page.click("button[type='submit']");

  await expect(page).toHaveURL("/dashboard");
});
```

------------------------------------------------------------------------

## STEP 7 : Run Tests

```bash
pnpm e2e
```

------------------------------------------------------------------------

## STEP 8 : Run with UI Mode

```bash
pnpm e2e:ui
```

------------------------------------------------------------------------

## STEP 9 : Run in Headed Mode

```bash
pnpm e2e:headed
```

------------------------------------------------------------------------

## STEP 10 : Debug Tests

```bash
pnpm e2e:debug
```

------------------------------------------------------------------------

## STEP 11 : View HTML Report

```bash
pnpm e2e:report
```

------------------------------------------------------------------------

## STEP 12 : Final Project Structure

```text
my-app/
│
├── src/
│
├── tests/
│   ├── home.spec.ts
│   ├── login.spec.ts
│   └── dashboard.spec.ts
│
├── playwright.config.ts
├── package.json
└── test-results/
```

------------------------------------------------------------------------

## WHAT PLAYWRIGHT TESTS

```text
Open Browser
      │
      ▼
Navigate to Website
      │
      ▼
Click Buttons
      │
      ▼
Fill Forms
      │
      ▼
Call Real Backend
      │
      ▼
Navigate Pages
      │
      ▼
Verify Final Result
```
