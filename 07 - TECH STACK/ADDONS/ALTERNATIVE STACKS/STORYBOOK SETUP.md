# STORYBOOK SETUP [ ALTERNATIVE STACKS ]

------------------------------------------------------------------------

Storybook is a workshop for building, documenting, and testing UI
components in isolation. Each component gets "stories" — rendered states
you develop and review without booting the whole app. It doubles as
living documentation and a visual-regression harness.

------------------------------------------------------------------------

## STEP 1 : Install Into A Next.js App

```bash
pnpm dlx storybook@latest init
```

The initializer detects Next.js + TypeScript, adds the
`@storybook/nextjs` framework, and creates a `.storybook/` config plus
example stories.

------------------------------------------------------------------------

## STEP 2 : Review The Config

```ts
// .storybook/main.ts
import type { StorybookConfig } from "@storybook/nextjs";

const config: StorybookConfig = {
  stories: ["../src/**/*.stories.@(ts|tsx)"],
  addons: [
    "@storybook/addon-essentials",
    "@storybook/addon-a11y",
    "@storybook/addon-interactions",
  ],
  framework: { name: "@storybook/nextjs", options: {} },
};
export default config;
```

------------------------------------------------------------------------

## STEP 3 : Write A Story

```tsx
// src/components/button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./button";

const meta: Meta<typeof Button> = {
  title: "UI/Button",
  component: Button,
  args: { children: "Click me" },
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Primary: Story = { args: { variant: "primary" } };
export const Disabled: Story = { args: { disabled: true } };
```

------------------------------------------------------------------------

## STEP 4 : Run The Workshop

```bash
pnpm storybook              # dev server at http://localhost:6006
pnpm build-storybook        # static site → storybook-static/
```

```text
   Component  ──▶  *.stories.tsx  ──▶  Storybook UI (6006)
        │               │                    │
     isolated       args/controls       a11y + interaction
      render        panel              tests per story
```

------------------------------------------------------------------------

## STEP 5 : Test Interactions And A11y

```tsx
// add a play function to drive and assert on a story
import { within, userEvent, expect } from "@storybook/test";

export const Clicks: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.click(canvas.getByRole("button"));
    await expect(canvas.getByText("Clicked")).toBeInTheDocument();
  },
};
```

The a11y addon flags contrast and ARIA issues live in the panel.

------------------------------------------------------------------------

## STEP 6 : Publish And Gate In CI

```yaml
# .github/workflows/storybook.yml
name: storybook
on: pull_request
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm build-storybook
      - run: pnpm dlx test-storybook --ci
```

Deploy `storybook-static/` to any static host for a shareable component
catalog. Windows note: run scripts from Git Bash or PowerShell — the CLI
is cross-platform and needs no extra tooling.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Colocate *.stories.tsx next to each component
✓ Cover every meaningful state as its own named story
✓ Use args + controls instead of hardcoding props
✓ Run the a11y addon and fix contrast/ARIA warnings
✓ Write play functions to test interactions automatically
✓ Build Storybook in CI so broken stories fail the PR
✓ Publish the static build as living component docs
✓ Mock providers/context in a decorator, not per story
✓ Keep stories free of real user data and secrets
```
