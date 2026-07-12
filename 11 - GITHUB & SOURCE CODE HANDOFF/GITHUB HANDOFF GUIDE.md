# GITHUB & SOURCE CODE HANDOFF [ CLEAN HANDS ]
------------------------------------------------------------------------

## OVERVIEW

This is the CLEAN-HANDS step. When it's done you no longer hold the
client's code or keys, and you're not linked to or responsible for their
infrastructure. You transfer the repo, hand over secrets safely, remove
yourself, rotate anything that passed through you, and keep only a
private archived copy for the warranty window. Do this right and a
breach on their side six months later has nothing to do with you.

------------------------------------------------------------------------

## HANDOFF FLOW

```text
   DURING BUILD            AT HANDOFF             CLEAN HANDS
  ┌──────────────┐   ┌──────────────────┐   ┌──────────────────┐
  │ private repo │   │ transfer repo to │   │ remove yourself  │
  │ ideally in   │──▶│ client's org     │──▶│ rotate keys      │
  │ client's org │   │ deliver env +    │   │ hand over access │
  │ from day one │   │ README + docs    │   │ archive → delete │
  └──────────────┘   └──────────────────┘   └──────────────────┘
```

------------------------------------------------------------------------

## DURING BUILD

```text
✓ Private repo from day one (never public with client work).
✓ Ideal: create it inside the CLIENT's GitHub org from the start,
  so ownership was never really yours to begin with.
✓ If you build in your own org, plan the transfer now — keep the
  history clean and commits professional.
```

------------------------------------------------------------------------

## AT HANDOFF — TRANSFER THE REPO

```text
□ GitHub → repo → Settings → scroll to "Danger Zone"
□ Transfer ownership → enter the client's org/username
□ Client accepts the transfer (they must have an org/account ready)
□ Repo now lives under the client — full history included
```

Transfer, don't just "add them as a collaborator." Ownership must
actually move to the client.

------------------------------------------------------------------------

## WHAT THE HANDOVER PACKAGE INCLUDES

```text
□ Source code (the transferred repo, full history)
□ .env.example  — every key, with placeholder values, committed
□ The REAL env values — handed over via password manager ONLY
    ✗ never paste secrets in chat, email, or a shared doc
□ README — install, run locally, env setup, deploy steps
□ Database migration files (versioned, in the repo)
□ docs/ + a plain-language client guide
```

Example `.env.example` (placeholders, safe to commit):

```text
DATABASE_URL="postgresql://user:pass@host:5432/db"
STRIPE_SECRET_KEY="sk_live_xxxxxxxxxxxx"
SENTRY_DSN="https://examplekey@o0.ingest.sentry.io/0"
NEXT_PUBLIC_APP_URL="https://example.com"
ADMIN_EMAIL="user@example.com"
```

------------------------------------------------------------------------

## CLEAN HANDS — REMOVE YOURSELF

```text
□ After transfer, remove yourself as a collaborator on the repo
□ Rotate ANY key/secret that ever passed through your hands:
    ✓ Stripe keys, DB passwords, API tokens, webhook secrets
    ✓ client resets them, or you rotate then hand over fresh values
□ Hand over ALL account access:
    ✓ hosting (Vercel), domain registrar, database, email, analytics
    ✓ client becomes owner; you are removed once verified
□ Keep ONE private archived copy for the warranty window only
    ✓ archived + private, used solely for bug fixes you owe
    ✓ delete it when the warranty period ends
□ Reconfirm your portfolio-rights clause (right to showcase the work)
```

------------------------------------------------------------------------

## WHY THIS PROTECTS YOU

```text
✓ You don't hold their code → not the source of a code leak.
✓ You don't hold their keys → not the source of a breach.
✓ You're removed from their accounts → no access, no blame.
✓ Rotated secrets → nothing you once saw still works.
Result: post-handoff, you are not liable for their systems.
Their infrastructure is theirs, cleanly.
```

------------------------------------------------------------------------

## TOOLS FOR THIS STEP

```text
GitHub .................. Settings → Transfer ownership
1Password / Bitwarden ... secure secrets handover (never chat)
Handoff checklist ....... the boxes above, ticked and dated
```

------------------------------------------------------------------------

## NEXT STEP

Code and access are cleanly with the client. Now sequence the money and
the final delivery — invoice first, transfer everything else on payment.

→ Continue to **12 - DELIVERY & FINAL PAYMENT**
