# DISCOVERY & REQUIREMENTS [ FIND THE REAL PROBLEM ]
------------------------------------------------------------------------

## WHY DISCOVERY EXISTS

------------------------------------------------------------------------

A client rarely knows what they need. They know what they *want* — "a
website," "something modern," "like this competitor." Your job in
discovery is to translate that into a build plan you can actually price,
scope, and ship.

Skip discovery and you get: scope creep, endless revisions, a client who
says "this isn't what I imagined," and a project that eats your margin.
Do discovery well and everything downstream (pricing, contract, tech,
build) gets easier because the decisions are already made.

```text
✓ Discovery ends with a WRITTEN scope doc + a sitemap. Nothing else.
✓ You cannot price what you have not scoped.
✓ "Understand the why" beats "collect a feature list."
```

------------------------------------------------------------------------

## THE DISCOVERY CALL

------------------------------------------------------------------------

Book 45-60 minutes. Record it (with permission) or take structured
notes. You are the interviewer — drive the conversation, do not just
take dictation.

Ask about GOALS, not features:

```text
□ Why do you want this site NOW? (What changed / what hurts?)
□ What does SUCCESS look like in 6 months? (leads? sales? bookings?)
□ Who are your target USERS? What do they need in 10 seconds?
□ What happens if we do nothing? (reveals real priority + budget)
□ Who are 2-3 competitors / sites you admire, and WHY?
□ What's the ONE thing this site must do well above all else?
```

Then the practical layer:

```text
□ How many PAGES roughly? (home, about, services, contact, blog...)
□ Which FEATURES? (forms, booking, e-commerce, chat, search, login)
□ Which INTEGRATIONS? (payments, CRM, email, analytics, calendar)
□ Who provides CONTENT — text, images, logo, brand? BY WHEN?
□ Any existing brand/design assets, or starting from scratch?
□ Hard DEADLINE and why? (event, launch, funding round?)
```

------------------------------------------------------------------------

## CONTENT IS THE #1 CAUSE OF DELAYS

------------------------------------------------------------------------

Say this out loud on the call: "The most common reason projects slip is
missing content." Then pin it down.

```text
✓ Name WHO delivers each piece of content, and the date they'll send it.
✓ "The client provides all copy and images by [date]" goes in the scope.
✓ If content is late, the timeline shifts — agree to this NOW, in writing.
✓ Offer copywriting / stock images as a PAID add-on if they can't deliver.
```

Never assume you'll write their copy for free. Never let a blank content
folder become your problem.

------------------------------------------------------------------------

## TURN FEATURES INTO DECISIONS → MAP TO TECH

------------------------------------------------------------------------

Every feature the client names is really a technical decision. Resolve
it here so the tech stack step is mechanical, not exploratory.

```text
FEATURE ASKED            →  DECISION           →  MAPS TO
-----------------------     ----------------      -------------------------
"a blog"                 →  CMS or MDX?        →  07 TECH STACK (CMS)
"contact form"           →  where do leads go? →  07 TECH STACK (forms/email)
"sell products"          →  full store?        →  07/ADDONS (e-commerce)
"live chat / support"    →  live vs async?     →  07/ADDONS (chat)
"search the site"        →  how many pages?    →  07/ADDONS (search)
"user accounts / login"  →  auth provider?     →  07 TECH STACK (auth)
"bookings / calendar"    →  self-serve?        →  07/ADDONS (scheduling)
"payments"               →  one-off or subs?   →  07 TECH STACK (payments)
```

Core categories (hosting, framework, CMS, auth, database, email, forms,
analytics) live in `07 - TECH STACK/`. Feature-specific tools
(e-commerce, chat, search, scheduling, etc.) live in
`07 - TECH STACK/ADDONS/`. You do not need to choose exact tools during
discovery — you need to KNOW WHICH CATEGORIES the project touches.

------------------------------------------------------------------------

## DEFINE SCOPE: IN vs OUT

------------------------------------------------------------------------

Scope is a fence. What's inside you build; what's outside is a change
request (= more money). Write both columns explicitly — the OUT column
prevents the arguments.

```text
IN SCOPE                          OUT OF SCOPE (change request / phase 2)
------------------------------    --------------------------------------
5 pages: home, about, 2x          Blog / news section
  service, contact                Online store / payments
Responsive (mobile + desktop)     User login / accounts
Contact form → client email       Multi-language / translations
Basic on-page SEO setup           Ongoing content writing
2 rounds of revisions             Logo / brand design
Client provides all copy+images   Custom illustrations / photography
```

```text
✓ Anything not written in the IN column is OUT by default.
✓ Vague words ("modern", "clean", "pop") are not scope — get examples.
✓ Every OUT item is a future upsell, not a loss.
```

------------------------------------------------------------------------

## DELIVERABLES OF THIS STEP

------------------------------------------------------------------------

You leave discovery with two documents. These become the backbone of
your proposal and contract.

```text
□ SCOPE DOCUMENT
    - Goals + definition of success
    - Target users
    - Page list
    - Feature list (with IN / OUT columns)
    - Integrations
    - Content responsibilities + dates
    - Assumptions + exclusions

□ SITEMAP + WIREFRAMES
    - Sitemap: the page tree and how pages connect
    - Wireframes: rough boxes-and-labels layout per key page
    - Low fidelity on purpose — structure, not visual design
```

Send the scope doc back to the client and get a written "yes, this is
correct" before you quote a price. That reply is your anchor if scope
creeps later.

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

------------------------------------------------------------------------

```text
REQUIREMENTS / SCOPE DOC   Notion  ·  Google Docs
WIREFRAMES + SITEMAP       Figma  ·  Excalidraw (fast, low-fidelity)
FEATURES CHECKLIST         A reusable checklist (Notion DB / spreadsheet)
CALL NOTES                 Notion  ·  Otter / built-in meeting notes
```

```text
✓ Keep a REUSABLE feature checklist — run every discovery call against it
  so you never forget to ask about auth, payments, SEO, analytics, etc.
✓ Excalidraw is perfect for sitemaps you sketch live on the call.
✓ Figma when the client wants to see something closer to real layout.
```

------------------------------------------------------------------------

## NEXT STEP

------------------------------------------------------------------------

You now know WHAT you're building and its exact scope. Time to put a
number on it. Continue to:

`04 - PRICING & PROPOSAL/PRICING GUIDE.md`
