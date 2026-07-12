# JEST SETUP [ UNIT TESTING ]
------------------------------------------------------------------------
Jest is a batteries-included unit-test runner: assertions, mocking, and
coverage in one package. Paired with React Testing Library (RTL) it tests
components the way users interact with them. In this stack Jest is the
*alternative* to the default Vitest setup — choose it for the mature
ecosystem, `next/jest` transform, and vast plugin/matcher catalogue.
------------------------------------------------------------------------
## STEP 1 : Install Jest + RTL
------------------------------------------------------------------------
```bash
pnpm add -D jest jest-environment-jsdom @types/jest \
  @testing-library/react @testing-library/jest-dom \
  @testing-library/user-event
```

Next.js ships a zero-config transform (`next/jest`) that reuses your
`next.config.js`, SWC, and path aliases — no Babel setup needed.
------------------------------------------------------------------------
## STEP 2 : Configure jest.config
------------------------------------------------------------------------
Create `jest.config.ts` at the project root.

```ts
import type { Config } from "jest";
import nextJest from "next/jest.js";

const createJestConfig = nextJest({ dir: "./" });

const config: Config = {
  testEnvironment: "jest-environment-jsdom",
  setupFilesAfterEnv: ["<rootDir>/jest.setup.ts"],
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
  },
  collectCoverageFrom: ["src/**/*.{ts,tsx}", "!src/**/*.d.ts"],
};

export default createJestConfig(config);
```

Load the custom matchers once in `jest.setup.ts`:

```ts
import "@testing-library/jest-dom";
```
------------------------------------------------------------------------
## STEP 3 : Wire pnpm scripts
------------------------------------------------------------------------
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage"
  }
}
```

On Windows `jest --watch` works in Git Bash, PowerShell, and cmd
identically; no extra shell config is required.
------------------------------------------------------------------------
## STEP 4 : Write a component test
------------------------------------------------------------------------
```tsx
// src/components/Counter.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import Counter from "./Counter";

test("increments when the button is clicked", async () => {
  const user = userEvent.setup();
  render(<Counter start={0} />);

  await user.click(screen.getByRole("button", { name: /increment/i }));

  expect(screen.getByText("1")).toBeInTheDocument();
});
```
------------------------------------------------------------------------
## STEP 5 : Test a pure utility
------------------------------------------------------------------------
```ts
// src/lib/slugify.test.ts
import { slugify } from "./slugify";

describe("slugify", () => {
  it("lowercases and hyphenates", () => {
    expect(slugify("Hello World")).toBe("hello-world");
  });
  it("strips punctuation", () => {
    expect(slugify("A, B & C!")).toBe("a-b-c");
  });
});
```
------------------------------------------------------------------------
## STEP 6 : How the pieces fit
------------------------------------------------------------------------
```text
              pnpm test
                  |
                jest
                  |
          next/jest transform (SWC)
                  |
        jsdom environment (fake DOM)
          |                    |
   React Testing Library   jest-dom matchers
          |                    |
   render + user-event    toBeInTheDocument()
                  |
          coverage report / CI gate
```
------------------------------------------------------------------------
## STEP 7 : Jest vs Vitest — when to choose
------------------------------------------------------------------------
```text
Choose JEST when:
  - You want the largest ecosystem of matchers/plugins/presets
  - The team already knows Jest APIs and snapshot workflows
  - A stable, slow-moving config is preferred over speed

Choose VITEST (the default here) when:
  - You want ESM-native, Vite-fast runs and HMR-style watch
  - Config should share the Vite/plugin pipeline you already use
  - In-source testing and instant startup matter most
```
APIs are near-identical (`describe`/`it`/`expect`), so migrating either
direction is cheap. Vitest stays the ( Default ); reach for Jest when
ecosystem maturity and `next/jest` integration are the priority.
------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Query by role/label/text — avoid getByTestId unless unavoidable
✓ Prefer userEvent over fireEvent for realistic interactions
✓ Never test implementation details; assert on rendered output
✓ Keep unit tests fast and isolated — mock network/db, not the DOM
✓ Use await + findBy* for async UI; never arbitrary setTimeout
✓ Co-locate *.test.tsx beside the source file it covers
✓ Gate CI on jest --coverage thresholds, not raw pass/fail
✓ Reserve Jest for units/components; leave E2E to Playwright/Cypress
```
