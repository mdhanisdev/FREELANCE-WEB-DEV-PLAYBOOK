# ADDONS [ ON-REQUEST FEATURES ]

Features you implement **only when the client specifically asks for them** —
not part of a normal build. Same structure as the core library: a **folder
per category**, the recommended tool has **`( Default )` in its filename**,
plus industry-standard alternatives.

Common baseline features (CMS, dates, carousel, bot protection, cookie
consent, uptime, dependency automation, bundle analysis) live at the **root**,
not here — see `00 - MASTER INDEX`.

------------------------------------------------------------------------

## CONTENT & DATA

```text
Category                    Tools  (★ = ( Default ))
─────────────────────────────────────────────────────────────
SEARCH SETUP/               ★ Meilisearch · Algolia · Typesense · Postgres-FTS
DATA TABLES SETUP/          ★ TanStack Table · AG Grid
CHARTS SETUP/               ★ Recharts · Tremor · Chart.js
PDF SETUP/                  ★ react-pdf · Puppeteer
```

------------------------------------------------------------------------

## ENGAGEMENT & MESSAGING

```text
LIVE CHAT & SUPPORT SETUP/  ★ Crisp · Intercom · Chatwoot · Tawk
NEWSLETTER & MARKETING/     ★ Loops · Mailchimp · ConvertKit · beehiiv
COMMENTS SETUP/             ★ giscus · Disqus
NOTIFICATIONS SETUP/        ★ Novu · Knock
PUSH NOTIFICATIONS SETUP/   ★ Web Push · OneSignal · Firebase FCM
REALTIME SETUP/             ★ Pusher · Ably · Supabase Realtime
```

------------------------------------------------------------------------

## UI & INTERACTION

```text
RICH TEXT EDITOR SETUP/     ★ TipTap · Lexical · BlockNote
DRAG & DROP SETUP/          ★ dnd-kit · Framer Motion Reorder
LIGHTBOX & GALLERY SETUP/   ★ yet-another-react-lightbox · PhotoSwipe
VIRTUALIZATION SETUP/       ★ TanStack Virtual · react-window
COMMAND MENU SETUP/         ★ cmdk (⌘K palette)
MEDIA PLAYER SETUP/         ★ Mux · react-player · Video.js
3D & WEBGL SETUP/           ★ React Three Fiber · Three.js · Spline
CALENDAR & SCHEDULING/      ★ FullCalendar · react-big-calendar · Cal.com
```

------------------------------------------------------------------------

## COMMERCE & COMMS

```text
E-COMMERCE SETUP/           ★ Shopify Hydrogen · Medusa · Snipcart · Saleor
SMS & OTP SETUP/            ★ Twilio · Vonage · AWS SNS
```

------------------------------------------------------------------------

## BACKEND / INFRA

```text
BACKGROUND JOBS SETUP/      ★ Inngest · Trigger.dev · BullMQ · Vercel Cron
AI SETUP/                   ★ Vercel AI SDK · LangChain · Mastra · pgvector
WEBHOOKS SETUP/             ★ Svix (send + receive/verify)
MULTI-TENANCY SETUP/        ★ shared-DB orgId + RBAC (SaaS teams)
MAPS SETUP/                 ★ Mapbox · Google Maps · Leaflet
I18N SETUP/                 ★ next-intl · next-i18next
PWA & OFFLINE SETUP/        ★ Serwist (service worker + manifest)
```

------------------------------------------------------------------------

## GROWTH & OPS

```text
FEATURE FLAGS SETUP/        ★ PostHog Flags · Flagsmith
LOAD TESTING SETUP/         ★ k6 · Artillery
DOCS SITE SETUP/            ★ Fumadocs · Nextra · Mintlify
```

------------------------------------------------------------------------

## ALTERNATIVE STACKS

```text
ALTERNATIVE STACKS/         Supabase · Convex · Expo (React Native)
                            · Turborepo · Storybook
```

> These replace parts of the core stack — read the "when to choose" section
> in each. (Astro & the other framework alternatives live in the core
> `FRAMEWORK SETUP/` folder.)

------------------------------------------------------------------------

## HOW TO CHOOSE

```text
Did the client specifically ask for this feature?
      │
      ├── No  → you probably don't need it (keep the build lean)
      │
      └── Yes → grab the ( Default ) file from the matching category,
                or an alternative if the project calls for it
```
