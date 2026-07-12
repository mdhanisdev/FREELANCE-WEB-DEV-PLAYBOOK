# MANTINE SETUP [ STYLING & UI / BATTERIES-INCLUDED COMPONENTS ]
------------------------------------------------------------------------
## STEP 1 : Install Core + Hooks

Mantine is a full component system with its own styling engine (no
Tailwind required). Start with the core and the hooks package.

```bash
pnpm add @mantine/core @mantine/hooks
```

Add PostCSS tooling Mantine uses for responsive mixins:

```bash
pnpm add -D postcss postcss-preset-mantine postcss-simple-vars
```

```js
// postcss.config.cjs
module.exports = {
  plugins: {
    "postcss-preset-mantine": {},
    "postcss-simple-vars": {
      variables: {
        "mantine-breakpoint-sm": "48em",
        "mantine-breakpoint-lg": "75em",
      },
    },
  },
};
```
------------------------------------------------------------------------
## STEP 2 : Provider in the App Router

Mantine needs its styles imported and a `ColorSchemeScript` injected in
`<head>` to prevent a flash of the wrong theme.

```tsx
// app/layout.tsx
import "@mantine/core/styles.css";
import {
  ColorSchemeScript,
  MantineProvider,
  mantineHtmlProps,
} from "@mantine/core";
import { theme } from "@/lib/theme";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" {...mantineHtmlProps}>
      <head>
        <ColorSchemeScript defaultColorScheme="auto" />
      </head>
      <body>
        <MantineProvider theme={theme} defaultColorScheme="auto">
          {children}
        </MantineProvider>
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 3 : Define a Theme

```ts
// lib/theme.ts
import { createTheme, type MantineColorsTuple } from "@mantine/core";

const brand: MantineColorsTuple = [
  "#eef3ff", "#dce4f5", "#b9c7e2", "#94a8d0", "#748dc0",
  "#5f7cb7", "#5474b4", "#44639f", "#3a588f", "#2d4b81",
];

export const theme = createTheme({
  primaryColor: "brand",
  colors: { brand },
  defaultRadius: "md",
  fontFamily: "Inter, sans-serif",
});
```
------------------------------------------------------------------------
## STEP 4 : Provider Composition

```text
   <html {...mantineHtmlProps}>
        │
        ├── <head> ── ColorSchemeScript  (sets data-mantine-color-scheme
        │                                  before paint → no FOUC)
        └── <body>
              └── MantineProvider(theme)
                     └── your routes / components
```
------------------------------------------------------------------------
## STEP 5 : Common Components

Everything is client-friendly and typed. A quick form example:

```tsx
"use client";
import { useState } from "react";
import { Button, Card, Group, Stack, TextInput, Title } from "@mantine/core";

export function SignupCard() {
  const [email, setEmail] = useState("user@example.com");

  return (
    <Card withBorder radius="md" padding="lg" maw={420}>
      <Stack>
        <Title order={3}>Create account</Title>
        <TextInput
          label="Email"
          value={email}
          onChange={(e) => setEmail(e.currentTarget.value)}
        />
        <Group justify="flex-end">
          <Button variant="default">Cancel</Button>
          <Button>Continue</Button>
        </Group>
      </Stack>
    </Card>
  );
}
```

Add optional packages as needs grow:

```bash
pnpm add @mantine/notifications @mantine/form @mantine/dates
```
------------------------------------------------------------------------
## STEP 6 : When to Choose Mantine

```text
Pick Mantine when you want:
  • 120+ components + 70+ hooks out of the box, minimal wiring
  • A native form/notifications/dates ecosystem (first-party)
  • Styling without committing to Tailwind utility classes

Prefer alternatives when:
  • You need copy-in, fully-owned source  → shadcn/ui
  • You are standardized on Material Design → MUI
  • Bundle size is critical and you use few widgets → headless + Tailwind
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Import @mantine/core/styles.css exactly once, in the root layout
✓ Always inject ColorSchemeScript in <head> to prevent theme flash
✓ Spread mantineHtmlProps on <html> for correct color-scheme handling
✓ Centralize brand tokens in createTheme, never hardcode hex in JSX
✓ Import styles.css from any extra package you add (e.g. notifications)
✓ Keep interactive components in "use client" files under App Router
✓ Use Mantine hooks (useDisclosure, useMediaQuery) instead of hand rolls
✓ Add feature packages lazily — don't install the whole suite upfront
```
