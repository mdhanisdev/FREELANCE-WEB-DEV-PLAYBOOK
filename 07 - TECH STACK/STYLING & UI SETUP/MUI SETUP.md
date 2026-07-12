# MUI SETUP [ STYLING & UI / MATERIAL DESIGN COMPONENTS ]
------------------------------------------------------------------------
## STEP 1 : Install Material UI + Emotion

MUI (Material UI) uses Emotion as its default styling engine. Install
the core, the icons, and the Next.js integration adapter.

```bash
pnpm add @mui/material @emotion/react @emotion/styled @mui/icons-material
pnpm add @mui/material-nextjs @emotion/cache
```
------------------------------------------------------------------------
## STEP 2 : App Router Integration

The App Router streams HTML, so Emotion's styles must be flushed with
the SSR stream. `@mui/material-nextjs` provides a cache provider that
handles this automatically — no manual registry needed.

```tsx
// app/layout.tsx
import { AppRouterCacheProvider } from "@mui/material-nextjs/v15-appRouter";
import { ThemeProvider, CssBaseline } from "@mui/material";
import { InitColorSchemeScript } from "@mui/material/styles";
import { theme } from "@/lib/theme";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <InitColorSchemeScript attribute="class" />
        <AppRouterCacheProvider options={{ key: "mui" }}>
          <ThemeProvider theme={theme}>
            <CssBaseline />
            {children}
          </ThemeProvider>
        </AppRouterCacheProvider>
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 3 : Define a Theme

Enable the built-in light/dark color schemes so MUI manages the toggle
via a CSS class and avoids a flash on load.

```ts
// lib/theme.ts
"use client";
import { createTheme } from "@mui/material/styles";
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"], display: "swap" });

export const theme = createTheme({
  cssVariables: { colorSchemeSelector: "class" },
  colorSchemes: { light: true, dark: true },
  typography: { fontFamily: inter.style.fontFamily },
  shape: { borderRadius: 10 },
  palette: { primary: { main: "#2d4b81" } },
});
```
------------------------------------------------------------------------
## STEP 4 : SSR Style Flow

```text
  Server render                Stream to client
 ┌───────────────┐            ┌───────────────────┐
 │ Emotion cache │  collect   │ <style> injected   │
 │ (key: "mui")  │ ─────────▶ │ inline, in order   │
 └───────────────┘            └───────────────────┘
        │                              │
   AppRouterCacheProvider        no FOUC / no hydration
   flushes on each chunk         style mismatch
```
------------------------------------------------------------------------
## STEP 5 : Using Components

Mark files that use interactive MUI components as client components.

```tsx
"use client";
import { useState } from "react";
import { Box, Button, Card, Stack, TextField, Typography } from "@mui/material";
import SaveIcon from "@mui/icons-material/Save";

export function SignupCard() {
  const [email, setEmail] = useState("user@example.com");

  return (
    <Card variant="outlined" sx={{ maxWidth: 420, p: 3 }}>
      <Stack spacing={2}>
        <Typography variant="h6">Create account</Typography>
        <TextField
          label="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          fullWidth
        />
        <Box sx={{ display: "flex", justifyContent: "flex-end", gap: 1 }}>
          <Button variant="text">Cancel</Button>
          <Button variant="contained" startIcon={<SaveIcon />}>
            Continue
          </Button>
        </Box>
      </Stack>
    </Card>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : When to Choose MUI

```text
Pick MUI when you want:
  • Strict Material Design 3 look and behavior
  • The largest, most mature enterprise component ecosystem
  • First-party data grid, date pickers, and charts (X packages)

Prefer alternatives when:
  • You want Tailwind-native, copy-in source → shadcn/ui
  • You dislike Emotion runtime CSS overhead → Tailwind / zero-runtime
  • You want batteries-included but non-Material → Mantine
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Wrap the app in AppRouterCacheProvider for correct SSR style flushing
✓ Keep createTheme in a "use client" module and import it in the layout
✓ Enable cssVariables + colorSchemes for flash-free dark mode
✓ Render <CssBaseline /> once to normalize browser defaults
✓ Add InitColorSchemeScript before content to set the scheme pre-paint
✓ Import icons individually (@mui/icons-material/Save) to keep bundles lean
✓ Prefer the sx prop or styled() over inline style objects
✓ Mark interactive component files "use client"; keep pages as Server Components
```
