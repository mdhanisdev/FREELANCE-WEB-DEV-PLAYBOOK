# MINTLIFY SETUP [ DOCS SITE ]

------------------------------------------------------------------------

Mintlify is a hosted documentation platform: you write MDX, Mintlify
builds and hosts a fast, branded docs site with search, analytics, and an
AI assistant. Use it when you want zero build/deploy ownership and a
best-in-class reading experience decoupled from your Next.js app.

------------------------------------------------------------------------

## STEP 1 : Install The CLI

```bash
pnpm add -g mint
```

The CLI gives you a local preview that matches the hosted render exactly.

------------------------------------------------------------------------

## STEP 2 : Create docs.json

The entire site is driven by one config file at the docs root.

```json
{
  "$schema": "https://mintlify.com/docs.json",
  "theme": "mint",
  "name": "Acme Docs",
  "colors": { "primary": "#0D9373" },
  "navigation": {
    "tabs": [
      {
        "tab": "Guides",
        "groups": [
          { "group": "Getting Started", "pages": ["index", "quickstart"] },
          { "group": "Guides", "pages": ["guides/auth", "guides/deploy"] }
        ]
      }
    ]
  }
}
```

------------------------------------------------------------------------

## STEP 3 : Lay Out Content

```text
   docs/
   ├── docs.json          → theme, nav, colors
   ├── index.mdx          → landing page
   ├── quickstart.mdx
   ├── guides/
   │   ├── auth.mdx
   │   └── deploy.mdx
   └── api-reference/
       └── openapi.json   → auto-generated API pages
```

```mdx
---
title: "Quickstart"
description: "Start here."
---

<Card title="Install" icon="download">
  Run the install command for your stack.
</Card>
```

------------------------------------------------------------------------

## STEP 4 : Preview Locally

```bash
cd docs
mint dev        # http://localhost:3000
```

On Windows, run this from Git Bash or PowerShell; the CLI needs Node 18+
on PATH. If port 3000 clashes with your Next.js app, pass `--port 3333`.

------------------------------------------------------------------------

## STEP 5 : Connect And Publish

```text
   GitHub repo (docs/) ──▶ Mintlify GitHub App ──▶ mintlify.app CDN
          │                        │                     │
      push to main           auto-build            custom domain
                             on every commit        docs.acme.com
```

Install the Mintlify GitHub App, point it at your docs folder, and every
push to `main` redeploys automatically — no CI to maintain.

------------------------------------------------------------------------

## STEP 6 : Generate API Docs

```bash
# scrape an OpenAPI spec into MDX pages
mint openapi-check openapi.json
```

Reference an `openapi.json` in navigation and Mintlify renders live,
try-it-out API pages with request/response samples.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep docs.json as the single source of nav truth
✓ Store docs in your repo so they version with code
✓ Let the GitHub App handle builds — avoid custom CI
✓ Generate API reference pages from an OpenAPI spec
✓ Set brand colors + logo for a native-feeling site
✓ Use built-in Card, Steps, and Accordion components
✓ Preview with mint dev before every push
✓ Point a custom domain at the hosted site for trust
✓ Choose Mintlify when you want to own content, not infra
```
