# TAILWIND SETUP [ STYLING & UI / UTILITY-FIRST CSS ]
------------------------------------------------------------------------
## STEP 1 : Install Tailwind v4 + PostCSS Plugin

Tailwind v4 ships a dedicated PostCSS plugin. No `tailwind.config.js` is
required to start — configuration lives in CSS.

```bash
pnpm add -D tailwindcss @tailwindcss/postcss postcss
```

Create `postcss.config.mjs` at the project root:

```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```
------------------------------------------------------------------------
## STEP 2 : Import Tailwind in globals.css

In v4 you replace the old `@tailwind base/components/utilities`
directives with a single `@import`.

```css
/* app/globals.css */
@import "tailwindcss";
```

Wire it into the root layout so every route inherits it:

```tsx
// app/layout.tsx
import "./globals.css";
import type { ReactNode } from "react";

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="min-h-dvh bg-background text-foreground antialiased">
        {children}
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 3 : Define Design Tokens as CSS Variables

Declare tokens under `:root`, then expose them to Tailwind with the
`@theme` directive so they become real utilities (`bg-background`).

```css
/* app/globals.css */
@import "tailwindcss";

:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.15 0 0);
  --primary: oklch(0.55 0.22 264);
  --radius: 0.625rem;
}

.dark {
  --background: oklch(0.15 0 0);
  --foreground: oklch(0.98 0 0);
  --primary: oklch(0.7 0.18 264);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --radius-lg: var(--radius);
}
```
------------------------------------------------------------------------
## STEP 4 : Token Resolution Flow

```text
  globals.css               @theme inline           Tailwind engine
 ┌──────────────┐          ┌──────────────┐        ┌───────────────┐
 │ :root vars   │ ──────▶  │ --color-*    │ ─────▶ │ bg-background │
 │ .dark vars   │          │ maps var()   │        │ text-primary  │
 └──────────────┘          └──────────────┘        └───────────────┘
        │                                                   │
        └────────── runtime theme switch (.dark) ───────────┘
```
------------------------------------------------------------------------
## STEP 5 : Add the cn() Helper

Combine `clsx` (conditional classes) with `tailwind-merge` (dedupes
conflicting utilities). This is the single merge point for class names.

```bash
pnpm add clsx tailwind-merge
```

```ts
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Usage inside a component:

```tsx
import { cn } from "@/lib/utils";

export function Badge({ active }: { active?: boolean }) {
  return (
    <span className={cn("rounded-lg px-2 py-1", active && "bg-primary text-white")}>
      Status
    </span>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Enforce Class Order with Prettier

`prettier-plugin-tailwindcss` sorts utility classes automatically on
save, keeping diffs clean across the team.

```bash
pnpm add -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc.json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./app/globals.css"
}
```

Run it across the repo (Windows-safe glob quoting):

```bash
pnpm exec prettier --write "**/*.{ts,tsx,css}"
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use a single @import "tailwindcss" — never mix v3 @tailwind directives
✓ Keep all design tokens in :root / .dark, expose them via @theme inline
✓ Prefer oklch() colors for perceptually uniform light/dark palettes
✓ Route every dynamic className through cn() to avoid utility conflicts
✓ Point tailwindStylesheet at globals.css so Prettier resolves tokens
✓ Set suppressHydrationWarning on <html> when toggling the .dark class
✓ Use logical/dvh units (min-h-dvh) for correct mobile viewport height
✓ Commit sorted classes — run prettier in CI to block unsorted diffs
```
