# PRODUCTION STACK [ SITUATION → TOOL INDEX ]

A reference library for building production-grade web apps as a freelancer.

Core stack assumed across all files:
**Next.js (App Router) · TypeScript · pnpm · Tailwind CSS**

------------------------------------------------------------------------

## HOW TO USE THIS LIBRARY

```text
Client Project
      │
      ▼
Pick the categories the project needs
      │
      ▼
Follow the matching SETUP.md file
      │
      ▼
Ship production-grade
```

Not every project needs every file. Start with **FOUNDATION**, then add
only what the project requires.

Each category is a **folder**; inside are tool-specific `SETUP.md` files.
The recommended default for average website work has **`( Default )` in
its filename** (and is marked ★ below); the rest are industry-standard
alternatives. Open a category folder, grab the `( Default )` file, or pick
the alternative that fits the project.

> Quick standard stack: take the `( Default )` file from each FOUNDATION +
> DATA & BACKEND folder and you have a production-grade baseline.

> This is **step 07** of the pathway. The business steps around it live
> one level up as numbered folders:
> `01 GET CLIENTS` → `02 COMMUNICATION` → `03 DISCOVERY` → `04 PRICING` →
> `05 CONTRACT` → `06 ACCOUNTS` → **07 TECH STACK (here)** → `08 BUILD` →
> `09 TESTING` → `10 LAUNCH` → `11 GITHUB HANDOFF` → `12 DELIVERY` →
> `13 MAINTENANCE`.  See `00 - README - THE ROADMAP.md` at the root.
>
> The stack builds the site; the surrounding steps get you the client,
> get you paid, and keep your hands clean.

------------------------------------------------------------------------

## FOUNDATION ( every project )

```text
Category folder            Tool files inside  (★ = recommended default)
─────────────────────────────────────────────────────────────
FRAMEWORK SETUP/           ★ Next.js · Remix · Astro · SvelteKit
STYLING & UI SETUP/        ★ Tailwind · shadcn/ui · Mantine · MUI
CODE QUALITY SETUP/        ★ ESLint · Prettier
GIT HOOKS SETUP/           ★ Husky · lint-staged
ENV VALIDATION SETUP/      ★ t3-env
TESTING SETUP/             ★ Vitest · ★ Playwright · Cypress · Jest
                           + a11y / lighthouse / cross-browser / security
```

------------------------------------------------------------------------

## DATA & BACKEND

```text
Category folder            Tool files inside  (★ = recommended default)
─────────────────────────────────────────────────────────────
DATABASE SETUP/            ★ PostgreSQL · MySQL · MongoDB · Redis
ORM SETUP/                 ★ Prisma · Drizzle · Mongoose
AUTH SETUP/                ★ Auth.js · Clerk · Better-Auth · Supabase-Auth
API SETUP/                 ★ Route Handlers · tRPC · GraphQL
FORMS & VALIDATION SETUP/  ★ React Hook Form · ★ Zod · Valibot
STATE MANAGEMENT SETUP/    ★ Zustand · Redux Toolkit · Jotai
DATA FETCHING SETUP/       ★ TanStack Query · SWR
```

------------------------------------------------------------------------

## PRODUCT FEATURES

```text
Category folder                Tool files inside  (★ = recommended)
─────────────────────────────────────────────────────────────
PAYMENTS SETUP/                ★ Stripe · Lemon Squeezy · Paddle · PayPal
EMAIL SETUP/                   ★ Resend · Postmark · SendGrid · Nodemailer
STORAGE SETUP/                 ★ UploadThing · AWS S3 · Cloudinary · R2
CACHING & RATE LIMITING SETUP/ ★ Upstash Redis · Rate Limiting
```

------------------------------------------------------------------------

## OPS & POLISH

```text
Category folder            Tool files inside  (★ = recommended)
─────────────────────────────────────────────────────────────
MONITORING SETUP/          ★ Sentry · Highlight
ANALYTICS SETUP/           ★ PostHog · ★ Vercel Analytics · GA4 · Plausible
LOGGING SETUP/             ★ Pino · Axiom
SEO SETUP/                 ★ Next.js Metadata (sitemap/robots/OG/JSON-LD)
ANIMATION SETUP/           ★ Framer Motion · GSAP
SECURITY SETUP/            ★ Security Headers · CSP Middleware
CI-CD SETUP/               ★ GitHub Actions · GitLab CI
HOSTING SETUP/             ★ Vercel · Netlify · Docker-VPS · Cloudflare · Railway
UPTIME MONITORING SETUP/   ★ BetterStack · UptimeRobot
DEPENDENCY AUTOMATION/     ★ Renovate · Dependabot
BUNDLE ANALYSIS SETUP/     ★ @next/bundle-analyzer
```

------------------------------------------------------------------------

## COMMON SITE FEATURES ( most builds )

Feature categories you reach for on the majority of sites — kept at root.

```text
Category folder            Tool files inside  (★ = ( Default ))
─────────────────────────────────────────────────────────────
CMS SETUP/                 ★ Sanity · Payload · Contentful · Strapi · MDX
DATE HANDLING SETUP/       ★ date-fns · Day.js · Luxon
CAROUSEL & SLIDER SETUP/   ★ Embla · Swiper · keen-slider
BOT PROTECTION SETUP/      ★ Cloudflare Turnstile · reCAPTCHA · hCaptcha
COOKIE CONSENT SETUP/      ★ vanilla-cookieconsent (GDPR)
```

------------------------------------------------------------------------

## ON-REQUEST FEATURES ( ADDONS/ )

Features you add **only when the client specifically asks**. Same structure
— a folder per category, `( Default )` file inside, plus alternatives.

```text
Content & data    SEARCH · DATA TABLES · CHARTS · PDF
Engagement        LIVE CHAT & SUPPORT · NEWSLETTER & MARKETING · COMMENTS
                  NOTIFICATIONS · PUSH NOTIFICATIONS · REALTIME
UI / interaction  RICH TEXT EDITOR · DRAG & DROP · LIGHTBOX & GALLERY
                  VIRTUALIZATION · COMMAND MENU · MEDIA PLAYER
                  3D & WEBGL · CALENDAR & SCHEDULING
Commerce & comms  E-COMMERCE · SMS & OTP
Backend / infra   BACKGROUND JOBS · AI · WEBHOOKS · MULTI-TENANCY
                  MAPS · I18N · PWA & OFFLINE
Growth            FEATURE FLAGS
Ops               LOAD TESTING · DOCS SITE
Alt stacks        ALTERNATIVE STACKS (Supabase · Convex · Expo · Turborepo · Storybook)

            → full list: ADDONS/00 - ADDONS INDEX.md
```

------------------------------------------------------------------------

## RECOMMENDED DEFAULT STACK ( 2026 )

```text
Framework          Next.js (App Router)
Language           TypeScript (strict)
Package Manager    pnpm
Styling            Tailwind CSS + shadcn/ui
Forms              React Hook Form + Zod
Client State       Zustand
Server State       TanStack Query
Database           PostgreSQL
ORM                Prisma (or Drizzle)
Auth               Auth.js (NextAuth v5)  /  Clerk
Payments           Stripe
Email              Resend + React Email
Uploads            UploadThing  /  AWS S3
Cache / Limits     Upstash Redis
Errors             Sentry
Analytics          PostHog + Vercel Analytics
Logging            Pino
Animation          Framer Motion
CI/CD              GitHub Actions
Hosting            Vercel  (Docker for self-host)
```

------------------------------------------------------------------------

## PROJECT DECISION FLOW

```text
Is it a static / marketing site?
      │
      ├── Yes → Framework + Styling & UI + SEO + Analytics + Hosting
      │
      └── No (full app)
              │
              ▼
      Add Database + ORM + Auth + API + Forms
              │
              ▼
      Needs money?      → Payments
      Needs email?      → Email
      Needs uploads?    → Storage
      Public traffic?   → Caching & Rate Limiting + Security
              │
              ▼
      Always: Monitoring + CI/CD + Hosting
```

------------------------------------------------------------------------

## GOLDEN RULES

```text
✓ Type-safe from database to UI
✓ Validate every input with Zod
✓ Keep secrets in env, never in code
✓ Automate quality gates (Husky + CI)
✓ Monitor production (errors + analytics)
✓ Ship small, ship often
✓ Only add a tool when the project needs it
```
