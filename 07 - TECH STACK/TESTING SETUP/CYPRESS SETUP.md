# CYPRESS SETUP [ E2E + COMPONENT TESTING ]
------------------------------------------------------------------------
Cypress is a browser-based test runner for end-to-end (E2E) and component
tests. It drives a real Chromium/Firefox/WebKit instance, so you assert on
the app exactly as a user sees it. In this stack it is the *alternative* to
the default Playwright setup — pick it when you want the interactive Test
Runner, time-travel debugging, and a gentler authoring curve.
------------------------------------------------------------------------
## STEP 1 : Install Cypress
------------------------------------------------------------------------
```bash
pnpm add -D cypress @testing-library/cypress start-server-and-test
```

First launch scaffolds the `cypress/` folder and opens the GUI:

```bash
pnpm exec cypress open
```

On Windows the binary is cached under
`%LOCALAPPDATA%\Cypress\Cache` (macOS/Linux use `~/.cache/Cypress`). If a
corporate proxy blocks the download, set `CYPRESS_INSTALL_BINARY` to a
mirror URL before running `pnpm install`.
------------------------------------------------------------------------
## STEP 2 : Configure Cypress
------------------------------------------------------------------------
Create `cypress.config.ts` at the project root. It defines both the E2E
and component runners in one file.

```ts
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    supportFile: "cypress/support/e2e.ts",
    specPattern: "cypress/e2e/**/*.cy.{ts,tsx}",
    video: false,
  },
  component: {
    devServer: {
      framework: "next",
      bundler: "webpack",
    },
    supportFile: "cypress/support/component.ts",
    specPattern: "src/**/*.cy.{ts,tsx}",
  },
});
```

Register the Testing Library matchers in the support files:

```ts
// cypress/support/e2e.ts  (and cypress/support/component.ts)
import "@testing-library/cypress/add-commands";
```
------------------------------------------------------------------------
## STEP 3 : Wire pnpm scripts
------------------------------------------------------------------------
`start-server-and-test` boots Next.js, waits for the port, runs the
suite, then tears the server down — identical on Windows and POSIX.

```json
{
  "scripts": {
    "cy:open": "cypress open",
    "cy:run": "cypress run",
    "cy:ct": "cypress run --component",
    "e2e": "start-server-and-test dev http://localhost:3000 cy:run"
  }
}
```
------------------------------------------------------------------------
## STEP 4 : Write an E2E spec
------------------------------------------------------------------------
```ts
// cypress/e2e/login.cy.ts
describe("Login flow", () => {
  it("signs a user in", () => {
    cy.visit("/login");
    cy.findByLabelText(/email/i).type("user@example.com");
    cy.findByLabelText(/password/i).type("hunter2{enter}");
    cy.location("pathname").should("eq", "/dashboard");
    cy.findByRole("heading", { name: /welcome/i }).should("be.visible");
  });
});
```
------------------------------------------------------------------------
## STEP 5 : Write a component spec
------------------------------------------------------------------------
Component tests mount a single React component in isolation — no server
required. Great for App Router client components.

```tsx
// src/components/Counter.cy.tsx
import Counter from "./Counter";

it("increments on click", () => {
  cy.mount(<Counter start={0} />);
  cy.findByRole("button", { name: /increment/i }).click();
  cy.findByText("1").should("exist");
});
```
------------------------------------------------------------------------
## STEP 6 : How the pieces fit
------------------------------------------------------------------------
```text
                 pnpm e2e
                     |
     start-server-and-test
        |                  \
   next dev (:3000)     cypress run
        |                     |
   App Router app  <----  browser driver
        |                     |
   real DOM / network   assertions + retries
                              |
                     screenshots / CI report
```
------------------------------------------------------------------------
## STEP 7 : Cypress vs Playwright — when to choose
------------------------------------------------------------------------
```text
Choose CYPRESS when:
  - You want the interactive GUI + time-travel snapshots
  - The team is new to E2E and values authoring speed
  - Component testing and E2E should share one tool/config

Choose PLAYWRIGHT (the default here) when:
  - You need true multi-tab / multi-origin / multi-browser
  - Parallel sharding across CI workers matters most
  - You want native WebKit + mobile emulation out of the box
```
Both are first-class; Playwright remains the ( Default ). Reach for
Cypress when developer experience and its component runner win the day.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Select by role/label (Testing Library) — never brittle CSS chains
✓ Keep specs independent; reset state via cy.request or task seeds
✓ Never use cy.wait(ms); rely on built-in retry-ability
✓ Run cy:run headless in CI, cy:open only for local debugging
✓ Disable video locally, enable artifacts (screenshots) in CI
✓ Gitignore cypress/videos and cypress/screenshots
✓ Pin the Cypress version; cache %LOCALAPPDATA%\Cypress on Windows CI
✓ Prefer component specs for logic, E2E for critical user journeys
```
