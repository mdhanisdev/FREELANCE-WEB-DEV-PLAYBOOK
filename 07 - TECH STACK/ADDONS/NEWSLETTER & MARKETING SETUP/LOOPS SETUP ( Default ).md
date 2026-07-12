# LOOPS SETUP [ NEWSLETTER & MARKETING ]
------------------------------------------------------------------------

Loops is the default for newsletter and marketing email in this stack:
a developer-first ESP with a clean REST API, contact properties, and
event-triggered campaigns. It is built for product marketing sends.

NOTE: newsletter / marketing email (broadcasts, drips, campaigns) is
DISTINCT from transactional email (receipts, password resets, OTPs).
Transactional lives in EMAIL SETUP. Keep the two flows separate so a
marketing unsubscribe never blocks a critical account email.

------------------------------------------------------------------------

## STEP 1 : Get an API key

1. Sign up at https://loops.so and create your mailing list.
2. Settings -> API -> Generate key.
3. Copy the key (starts with a long token).

------------------------------------------------------------------------

## STEP 2 : Store the key (server-only)

```bash
# .env.local
LOOPS_API_KEY="loops-server-side-secret"
```

Never prefix marketing keys with `NEXT_PUBLIC_`; all sends run
server-side. Windows: edit in VS Code (UTF-8), not Notepad.

------------------------------------------------------------------------

## STEP 3 : Install the SDK

```bash
pnpm add loops
```

------------------------------------------------------------------------

## STEP 4 : Create a typed client

```ts
// lib/loops.ts
import { LoopsClient } from "loops";

export const loops = new LoopsClient(process.env.LOOPS_API_KEY!);
```

------------------------------------------------------------------------

## STEP 5 : Subscribe from a Route Handler

```ts
// app/api/newsletter/route.ts
import { NextResponse } from "next/server";
import { loops } from "@/lib/loops";

export async function POST(req: Request) {
  const { email } = (await req.json()) as { email: string };

  await loops.createContact(email, {
    source: "website-footer",
    subscribed: true, // marketing consent, not transactional
  });

  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 6 : Trigger a campaign with an event

```ts
await loops.sendEvent({
  email: "user@example.com",
  eventName: "signup_completed",
});
```

------------------------------------------------------------------------

## STEP 7 : Channel separation

```text
Marketing (Loops)                 Transactional (EMAIL SETUP)
+---------------------+           +---------------------------+
| broadcasts / drips  |           | receipts / password reset |
| honors unsubscribe  |           | always delivered          |
| LOOPS_API_KEY       |           | separate provider + key   |
+---------------------+           +---------------------------+
        \_______________ never share keys _______________/
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep LOOPS_API_KEY server-only; call the API from Route Handlers
✓ Separate marketing (Loops) from transactional email entirely
✓ Store explicit marketing consent; honor unsubscribe automatically
✓ Use events, not per-send code, to trigger drip campaigns
✓ Validate and normalize email before createContact
✓ Rate-limit the subscribe endpoint to block spam signups
✓ Keep Loops as the default developer-friendly marketing ESP
```
