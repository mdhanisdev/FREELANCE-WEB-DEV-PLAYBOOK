# SHADCN-UI SETUP [ STYLING & UI / COPY-IN COMPONENT LIBRARY ]
------------------------------------------------------------------------
## STEP 1 : Prerequisites

shadcn/ui is not an installed dependency — it copies component source
into your repo. It requires Tailwind and the `cn()` helper already set
up (see TAILWIND SETUP.md).

```bash
pnpm add clsx tailwind-merge class-variance-authority
```
------------------------------------------------------------------------
## STEP 2 : Initialize

Run the CLI. It detects Next.js + Tailwind, writes `components.json`,
and patches `globals.css` with the base theme tokens.

```bash
pnpm dlx shadcn@latest init
```

Answer the prompts:

```text
✔ Which style?            New York
✔ Base color?            Neutral
✔ CSS variables?         Yes
```
------------------------------------------------------------------------
## STEP 3 : components.json

The generated manifest tells the CLI where to place files and how to
resolve aliases. Keep it committed.

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui"
  }
}
```
------------------------------------------------------------------------
## STEP 4 : Add Components

Each command copies fully-owned, editable source into
`components/ui/`. Add only what you use.

```bash
pnpm dlx shadcn@latest add button card input dialog dropdown-menu
```

```text
components/
└── ui/
    ├── button.tsx      ← yours to edit, no black box
    ├── card.tsx
    ├── input.tsx
    ├── dialog.tsx
    └── dropdown-menu.tsx
```

Use it like any local component:

```tsx
import { Button } from "@/components/ui/button";

export default function Page() {
  return <Button variant="outline">Continue</Button>;
}
```
------------------------------------------------------------------------
## STEP 5 : Dark Mode with next-themes

```bash
pnpm add next-themes
```

```tsx
// components/theme-provider.tsx
"use client";
import { ThemeProvider as NextThemesProvider } from "next-themes";
import type { ReactNode } from "react";

export function ThemeProvider({ children }: { children: ReactNode }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </NextThemesProvider>
  );
}
```

Wrap the app and suppress the hydration warning from the `class` swap:

```tsx
// app/layout.tsx
import { ThemeProvider } from "@/components/theme-provider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Toasts (sonner) + Icons (lucide)

sonner is the recommended toast primitive. lucide-react ships as a
shadcn dependency and is used throughout the components.

```bash
pnpm dlx shadcn@latest add sonner
pnpm add lucide-react
```

Mount the `<Toaster />` once, near the root, then fire toasts anywhere:

```tsx
// app/layout.tsx (inside <body>)
import { Toaster } from "@/components/ui/sonner";
// ...
<ThemeProvider>
  {children}
  <Toaster richColors position="top-right" />
</ThemeProvider>
```

```tsx
"use client";
import { toast } from "sonner";
import { Save } from "lucide-react";
import { Button } from "@/components/ui/button";

export function SaveButton() {
  return (
    <Button onClick={() => toast.success("Saved", { description: "user@example.com" })}>
      <Save className="mr-2 size-4" /> Save
    </Button>
  );
}
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Treat components/ui/* as owned source — edit it, don't fight it
✓ Add components on demand; never bulk-add the whole registry
✓ Keep cssVariables:true so theming flows through CSS tokens
✓ Use attribute="class" with next-themes to match .dark tokens
✓ Set suppressHydrationWarning on <html> to silence theme flicker warns
✓ Mount a single <Toaster /> at the root, never per-page
✓ Standardize on lucide-react; size icons with size-4 utilities
✓ Re-run `add` after registry updates, then review the diff before commit
```
