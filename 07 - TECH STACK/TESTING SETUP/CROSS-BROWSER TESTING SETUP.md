# CROSS-BROWSER TESTING SETUP [ PLAYWRIGHT ]

------------------------------------------------------------------------

## STEP 1 : Install Playwright

```bash
pnpm add -D @playwright/test
pnpm exec playwright install
```

------------------------------------------------------------------------

## STEP 2 : Configure Browsers

`playwright.config.ts`

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",

  projects: [
    {
      name: "Chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "Firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "WebKit",
      use: { ...devices["Desktop Safari"] },
    },
  ],

  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: true,
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Create Test

`tests/home.spec.ts`

```ts
import { test, expect } from "@playwright/test";

test("homepage loads", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveTitle(/Next/i);
});
```

------------------------------------------------------------------------

## STEP 4 : Run Tests

```bash
pnpm exec playwright test
```

------------------------------------------------------------------------

## STEP 5 : Final Project Structure

```text
tests/
├── home.spec.ts

playwright.config.ts
package.json
```
