# NEXT-BUNDLE-ANALYZER SETUP [ BUNDLE ANALYSIS ]

------------------------------------------------------------------------

`@next/bundle-analyzer` wraps your Next.js config and emits interactive
treemaps of every client, server, and edge bundle so you can catch
regressions before they ship. This is the default profiler for the App
Router stack.

------------------------------------------------------------------------

## STEP 1 : Install The Analyzer

```bash
pnpm add -D @next/bundle-analyzer
```

The package is dev-only; it never touches your production runtime and is
tree-shaken out of standard builds.

------------------------------------------------------------------------

## STEP 2 : Wrap next.config.ts

```ts
// next.config.ts
import type { NextConfig } from "next";
import withBundleAnalyzer from "@next/bundle-analyzer";

const withAnalyzer = withBundleAnalyzer({
  enabled: process.env.ANALYZE === "true",
  openAnalyzer: true,
});

const nextConfig: NextConfig = {
  reactStrictMode: true,
  experimental: { optimizePackageImports: ["lucide-react"] },
};

export default withAnalyzer(nextConfig);
```

The wrapper only activates when `ANALYZE=true`, so day-to-day builds stay
fast and unchanged.

------------------------------------------------------------------------

## STEP 3 : Add A Reusable Script

```json
{
  "scripts": {
    "build": "next build",
    "analyze": "cross-env ANALYZE=true next build"
  }
}
```

```bash
pnpm add -D cross-env
```

`cross-env` normalizes environment-variable syntax across shells. On
Windows PowerShell, `ANALYZE=true next build` fails without it because
inline `VAR=value` assignment is a POSIX-only feature.

------------------------------------------------------------------------

## STEP 4 : Run And Read The Report

```bash
pnpm analyze
```

Three HTML files open in your browser from `.next/analyze/`:

```text
   .next/analyze/
   ├── client.html   → shipped to the browser  (watch this first)
   ├── nodejs.html   → server components + RSC payloads
   └── edge.html     → middleware + edge routes

   [ Treemap legend ]
   ┌───────────────────────────────┐
   │  ██ Stat size    (raw source) │
   │  ██ Parsed size  (minified)   │
   │  ██ Gzip size    (over wire)  │  ← optimize against THIS
   └───────────────────────────────┘
```

Always judge weight by gzip size, since that is what users download.

------------------------------------------------------------------------

## STEP 5 : Act On The Findings

- Replace heavy libs (`moment` → `date-fns`, `lodash` → `lodash-es`).
- Push large client components behind `next/dynamic` with `ssr: false`.
- Add offenders to `optimizePackageImports` for automatic barrel pruning.

```ts
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("@/components/heavy-chart"), {
  ssr: false,
  loading: () => <p>Loading chart…</p>,
});
```

------------------------------------------------------------------------

## STEP 6 : Gate Bundle Size In CI

```yaml
# .github/workflows/bundle.yml
name: bundle
on: pull_request
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm analyze
```

Pair with a size-limit action to fail PRs that exceed a byte budget.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Optimize against gzip size, not raw stat size
✓ Keep ANALYZE gated behind an env flag so normal builds stay fast
✓ Inspect client.html first — it is what users actually download
✓ Use cross-env for shell parity between Windows and Unix
✓ Prefer dynamic imports for below-the-fold heavy widgets
✓ Add barrel-heavy packages to optimizePackageImports
✓ Track bundle size in CI with a hard byte budget
✓ Re-run analyze after every dependency upgrade
✓ Never ship the analyzer output folder to production
```
