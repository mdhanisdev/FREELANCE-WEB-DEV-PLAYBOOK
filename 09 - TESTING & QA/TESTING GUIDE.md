# TESTING & QA [ TEST BEFORE YOU HAND OVER ]
------------------------------------------------------------------------

## WHERE YOU ARE

The build from **08 - BUILD & DEV WORKFLOW/** is feature-complete. Do not
launch yet. This step is your safety check: catch the broken form, the
inaccessible button, and the slow page **before** the client — or worse,
a real user — finds it.

```text
08 (build) ──▶ 09 (test & QA) ──▶ 10 (launch)
                    ▲ you are here
```

------------------------------------------------------------------------

## WHAT TO TEST, AND WITH WHAT

The setup for each of these tools lives in
**07 - TECH STACK/TESTING SETUP/**. This guide is about *using* them
before handover.

```text
LAYER            TOOL           WHAT IT CATCHES
---------------  -------------  ----------------------------------------
Unit / component Vitest         Broken logic, component regressions
End-to-end       Playwright     Whole user flows failing (sign up,
                                 checkout, contact)
Accessibility    axe / a11y     Missing labels, contrast, keyboard traps
Performance      Lighthouse     Slow loads, layout shift, big bundles
Cross-browser    Playwright /   "Works on my Chrome" bugs in Safari,
                 BrowserStack    Firefox, mobile
Security         pnpm audit     Vulnerable dependencies
```

------------------------------------------------------------------------

## THE TESTING PYRAMID

```text
              ╱‾‾‾‾‾‾‾╲
             ╱   E2E   ╲        few — Playwright, full user journeys
            ╱───────────╲
           ╱  COMPONENT  ╲      some — Vitest, rendered components
          ╱───────────────╲
         ╱      UNIT       ╲    many — Vitest, pure logic / utils
        ╱___________________╲

  Many fast unit tests at the base, fewer slow e2e tests at the top.
```

------------------------------------------------------------------------

## UNIT & COMPONENT — VITEST

```text
□ Test pure functions: formatting, validation, calculations
□ Test components render and respond to props / state
□ Test edge cases: empty, null, huge, wrong type
□ Run in watch mode while building; run the full suite before push
□ Aim for meaningful coverage of logic, not a 100% vanity number
```

------------------------------------------------------------------------

## END-TO-END — PLAYWRIGHT

Drive the real app in a real browser. Cover the flows that, if broken,
make the site worthless.

```text
□ Home loads and primary nav works
□ Contact / lead form: fill → submit → success state shown
□ Auth flow (if any): sign up → sign in → sign out
□ Checkout / payment (if any): happy path end to end
□ 404 and error pages render correctly
```

------------------------------------------------------------------------

## ACCESSIBILITY — axe / a11y

```text
✓ All images have meaningful alt text (or empty alt if decorative)
✓ Every form field has an associated label
✓ Text meets WCAG AA contrast against its background
✓ The whole site is usable by keyboard alone (visible focus states)
✓ Headings are in order (one h1, logical h2/h3 nesting)
✓ Interactive elements are real buttons/links, not bare divs
```

Run axe in your Playwright tests so accessibility is checked on every
run, not just once by hand.

------------------------------------------------------------------------

## PERFORMANCE — LIGHTHOUSE

```text
□ Run Lighthouse on key pages (home, a content page, a form page)
□ Target strong scores: Performance, Accessibility, Best Practices, SEO
□ Optimize images (next/image, correct sizes, modern formats)
□ Check Core Web Vitals: LCP, CLS, INP
□ Watch bundle size — code-split heavy client components
```

------------------------------------------------------------------------

## CROSS-BROWSER & RESPONSIVE

```text
□ Chrome, Firefox, Safari (desktop)
□ Mobile Safari (iOS) and Chrome (Android)
□ Layout holds at mobile, tablet, and desktop widths
□ Touch targets are big enough; no hover-only interactions on mobile
□ Use Playwright projects for engines; BrowserStack for real devices
```

------------------------------------------------------------------------

## SECURITY

```text
✓ pnpm audit is clean — no known high/critical vulnerabilities
✓ No secrets in the repo or client bundle (env validation from 07)
✓ Security headers set (CSP, HSTS, X-Content-Type-Options)
✓ All inputs validated/sanitized server-side
✓ Dependencies reasonably up to date
```

------------------------------------------------------------------------

## PRE-HANDOVER QA CHECKLIST

The final human pass. Walk the real site as a picky first-time visitor.

```text
□ Responsive: looks right on mobile, tablet, and desktop
□ Forms work END TO END: submit → email and/or DB record actually
  received (check the inbox and the database, not just the UI toast)
□ 404 page is designed and on-brand
□ Error / 500 page is designed and on-brand
□ No broken links (internal and external)
□ Real content in place — no "lorem ipsum", no placeholder images
□ No errors or warnings in the browser console
□ Favicon, page titles, meta descriptions, and OG images set
□ pnpm audit is clean
□ Lighthouse run and scores acceptable on key pages
□ Tested in Chrome, Firefox, and Safari
```

Only when every box is checked is the site ready to launch.

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

- **Vitest** — unit and component tests, fast watch mode.
- **Playwright** — end-to-end flows and cross-engine testing.
- **axe / a11y** — automated accessibility checks (run inside Playwright).
- **Lighthouse** — performance, SEO, and best-practices audits.
- **BrowserStack** — real device and browser matrix beyond emulation.
- **pnpm audit** — dependency vulnerability scan.
- **07 - TECH STACK/TESTING SETUP/** — the config that stands all of the
  above up in the project.

------------------------------------------------------------------------

## NEXT STEP

Everything is tested, accessible, fast, and QA-approved. Time to go live.

➡  **10 - LAUNCH/**  — final pre-flight, production deploy, and go-live.
