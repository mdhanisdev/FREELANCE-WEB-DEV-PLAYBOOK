# ACCOUNTS & OWNERSHIP [ WHO HOLDS THE KEYS ]
------------------------------------------------------------------------

## THE ONE RULE

Anything that holds the client's **money, data, domain, or legal
identity** must live in the **CLIENT's own account**. Your craft tooling
(the stuff you use to build) stays yours.

If it can bill a card, receive a payout, store user data, control the
domain, or represent the business legally — it belongs to the client. No
exceptions, no "I'll just use mine for now."

```text
✓ Client money / data / domain / legal identity ...... CLIENT account
✓ Your craft tooling (editor, design, PM) ............ YOUR account
✓ When in doubt, ask: "If we part ways tomorrow, who
  is locked out?"  The client must never be locked out.
```

------------------------------------------------------------------------

## WHY THIS MATTERS

- **Clean handoff.** If every business asset is already in the client's
  account, handoff is just removing yourself — nothing to migrate.
- **Legal / liability.** Stripe under your name means their revenue runs
  through your KYC and your bank. That is your tax problem and your
  liability. Never do it.
- **Trust.** Clients relax when they own everything from day one. It also
  protects you: you are not the single point of failure.
- **No hostage situations.** You never want to be the person holding a
  domain the client "can't get back."

------------------------------------------------------------------------

## CLIENT MUST OWN

```text
ASSET                     SERVICE (example)      WHY IT'S THE CLIENT'S
------------------------  ---------------------  -----------------------
Domain registrar          Namecheap / Cloudflare Their brand & identity
DNS                       Cloudflare / registrar Controls the whole site
Hosting / deploy          Vercel                 Runs their product
Database                  Neon / Supabase        Holds their user data
Payments                  Stripe                 Their bank + KYC —
                                                  NEVER your name
Transactional email       Resend (verified       Sends as their domain
                          domain)
Analytics                 Plausible / GA4        Their traffic data
Error monitoring          Sentry                 Their production logs
Search Console            Google Search Console   Their SEO / indexing
3rd-party API keys        OpenAI, Maps, etc.     Billed to their card
CMS                       Sanity / Contentful    Their content
Social accounts           X, Instagram, etc.     Their audience
```

The **Stripe** line is the one people get wrong. Stripe is tied to a
legal entity, a bank account, and identity verification (KYC). It MUST
be the client's — created by them, verified with their documents, paying
out to their bank. You may be invited as a team member to help set it
up, but the account is theirs.

------------------------------------------------------------------------

## YOU OWN

```text
ASSET                     SERVICE (example)      NOTE
------------------------  ---------------------  -----------------------
Source control (build)    Your GitHub            During build only; see
                                                  handoff note below
Design files              Figma                  Your craft; export what
                                                  the contract requires
Project management        Linear / Notion        Your process
Invoicing / accounting    Wave / QuickBooks      Your business
Password manager          1Password / Bitwarden  Your vault
Portfolio                 Your own site          Showcase (per contract)
```

Note on GitHub: it is fine to build in your own repo, but the **best
practice is the client's org from day one** (see the build workflow
guide). Either way, the code ownership transfers at handoff per the
contract.

------------------------------------------------------------------------

## BEST PRACTICE: CLIENT CREATES, INVITES YOU

The cleanest pattern for every business asset:

```text
□ Client creates the account (their email, their card, their identity)
□ Client invites YOU as a member / collaborator / developer
□ You do the setup and build work inside their account
□ At handoff you simply LEAVE the account — nothing to migrate
```

Compare with the messy pattern (avoid):

```text
✗ You create everything under your accounts "to move fast"
✗ At handoff you must migrate domains, transfer Stripe, re-verify
  email domains, move databases, reset billing — days of risky work,
  and the client is exposed the whole time
```

Ten minutes of the client clicking "create account" up front saves days
of migration pain later.

------------------------------------------------------------------------

## HANDLING SECRETS

```text
✓ Share credentials ONLY through a password manager vault
✓ Never send passwords, API keys, or tokens over email or chat
✓ Never commit secrets to git (env validation guards this later)
✓ Give each service its own strong, unique password
✓ Turn on 2FA on every money/data/domain account
```

**Rotate anything that passed through you.** If a key, password, or
token ever touched your machine, chat, or clipboard, rotate it at
handoff so the client's live secrets are ones you have never seen.

```text
AT HANDOFF — SECRET HYGIENE
□ Remove yourself from every client account
□ Rotate API keys you generated or used
□ Reset any shared passwords
□ Confirm client 2FA is on and recovery codes are theirs
```

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

- **1Password** or **Bitwarden** — shared vault for handing over secrets
  safely; never paste credentials into email or chat.
- **Accounts checklist** — the CLIENT MUST OWN table above, turned into a
  live tracker (account created? you invited? 2FA on?).
- **Registrar / Vercel / Stripe / Neon dashboards** — set up under the
  client's login, with you invited as a member.

------------------------------------------------------------------------

## NEXT STEP

Ownership is settled and the client holds the keys. Now choose the tools
you will actually build with and stand up the project.

➡  **07 - TECH STACK/**  — pick your stack (Next.js, database, testing,
   code quality) and scaffold the project.
