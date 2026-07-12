# LAUNCH [ GO LIVE ]
------------------------------------------------------------------------

## OVERVIEW

Launch is a gate, not a button. You do not "push to prod and hope." You
pass a pre-launch checklist, flip the domain, verify the LIVE site with
real inputs, then watch it for 24-48 hours. Nothing here is optional —
a broken payment form on launch day is the fastest way to lose a client
and a testimonial.

------------------------------------------------------------------------

## THE LAUNCH FLOW

```text
   PRE-LAUNCH GATE          GO-LIVE            POST-LAUNCH WATCH
  ┌───────────────┐   ┌────────────────┐   ┌──────────────────┐
  │ build clean   │   │ point domain   │   │ Sentry: 24-48h   │
  │ Lighthouse ✓  │──▶│ verify HTTPS   │──▶│ uptime monitor   │
  │ headers ✓     │   │ prod env vars  │   │ watch first users│
  │ Sentry live   │   │ submit sitemap │   │ hotfix if needed │
  │ clean test    │   │ test LIVE site │   │                  │
  └───────────────┘   └────────────────┘   └──────────────────┘
```

------------------------------------------------------------------------

## PRE-LAUNCH GATE (must all pass)

```text
□ Production build passes clean locally
    npm run build   → zero errors, zero type errors
□ Lighthouse green on all four (mobile profile):
    ✓ Performance      ≥ 90
    ✓ SEO              ≥ 90
    ✓ Best Practices   ≥ 90
    ✓ Accessibility    ≥ 90
□ Security headers on (next.config.js / middleware):
    ✓ Strict-Transport-Security
    ✓ X-Content-Type-Options: nosniff
    ✓ X-Frame-Options / frame-ancestors
    ✓ Content-Security-Policy (at least a baseline)
□ Error tracking (Sentry) live and receiving events
    → trigger a test error, confirm it lands in the dashboard
□ Analytics firing on real page views (GA4 / Plausible / Vercel)
□ Remove ALL test / example / debug artifacts:
    ✓ delete /test, /example, /debug, /_dev pages
    ✓ strip console.log / console.debug from shipped code
    ✓ remove seed/demo data and dummy accounts
□ Database migrations DEPLOYED, not pushed:
    ✓ prisma migrate deploy   (production-safe, versioned)
    ✗ NOT prisma db push      (dev-only, can drop columns)
```

------------------------------------------------------------------------

## WHY db:deploy, NOT db:push

```text
✓ db:deploy → applies committed migration files in order,
              reproducible, safe on live data, auditable.
✗ db:push   → force-syncs schema to the DB, can silently drop
              columns/tables and destroy production data.
```

Push is a dev convenience. Deploy is what production runs.

------------------------------------------------------------------------

## GO-LIVE STEPS

```text
□ Point the REAL domain at the deployment
    → add domain in Vercel → update DNS (A / CNAME) at registrar
□ Verify HTTPS / SSL
    → cert issued, green padlock, http:// redirects to https://
□ Confirm PRODUCTION env vars are set (not just Preview):
    ✓ Vercel → Project → Settings → Environment Variables
    ✓ every secret exists under the "Production" scope
    ✓ live keys, not test keys (Stripe live, real DB URL)
□ Submit sitemap to Google Search Console
    → verify domain ownership → Sitemaps → submit /sitemap.xml
```

A common miss: env vars set only for Preview. The preview build works,
the production domain 500s. Check the scope column explicitly.

------------------------------------------------------------------------

## TEST THE LIVE SITE (not preview — the real domain)

```text
□ Forms: submit each one, confirm it stores + emails you
□ Payments: run a REAL card through a live checkout
    → then refund it; confirm webhook + order record
□ Login / signup / password reset end to end
□ Every key user flow the client cares about
□ 404 and error pages render correctly
□ Mobile: open on an actual phone, not just devtools
```

------------------------------------------------------------------------

## POST-LAUNCH WATCH (24-48h)

```text
□ Sentry open — triage any new errors immediately
□ Uptime monitor (BetterStack) pinging the domain
    → alert to email/SMS on downtime
□ Watch the first real users move through the site
□ Keep a hotfix branch ready; deploy fixes fast
✓ Do not disappear the moment DNS resolves — the first two
  days surface the bugs that testing missed.
```

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

```text
Vercel .................. hosting, domains, env vars, deploys
Google Search Console ... sitemap submission, indexing
Sentry .................. error tracking + alerts
BetterStack ............. uptime monitoring + downtime alerts
Lighthouse .............. perf / SEO / a11y / best-practices audit
```

------------------------------------------------------------------------

## NEXT STEP

The site is live and stable. Now hand over the code cleanly so you carry
no ongoing liability.

→ Continue to **11 - GITHUB & SOURCE CODE HANDOFF**
