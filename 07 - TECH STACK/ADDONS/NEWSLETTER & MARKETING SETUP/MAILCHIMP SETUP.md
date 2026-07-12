# MAILCHIMP SETUP [ NEWSLETTER & MARKETING ]
------------------------------------------------------------------------

Mailchimp is a mature marketing platform with audiences, segments, and
a drag-and-drop campaign builder. Choose it for non-technical marketing
teams. This guide wires audience signups into Next.js App Router.

NOTE: this is MARKETING email (newsletters, campaigns), which is
DISTINCT from transactional email (receipts, resets) in EMAIL SETUP.
Mailchimp's Transactional product (Mandrill) is a separate add-on; do
not route account-critical mail through your marketing audience.

------------------------------------------------------------------------

## STEP 1 : Collect credentials

1. Sign in at https://mailchimp.com.
2. Account -> Extras -> API keys: create a key. The suffix after the
   dash is your server prefix (e.g. `us21`).
3. Audience -> Settings -> copy the `Audience ID` (List ID).

------------------------------------------------------------------------

## STEP 2 : Store configuration (server-only)

```bash
# .env.local
MAILCHIMP_API_KEY="xxxxxxxxxxxxxxxxxxxxxxxx-us21"
MAILCHIMP_SERVER_PREFIX="us21"
MAILCHIMP_AUDIENCE_ID="a1b2c3d4e5"
```

All values are secret. Windows: save `.env.local` as UTF-8 via VS Code.

------------------------------------------------------------------------

## STEP 3 : Install the SDK

```bash
pnpm add @mailchimp/mailchimp_marketing
```

------------------------------------------------------------------------

## STEP 4 : Configure the client

```ts
// lib/mailchimp.ts
import mailchimp from "@mailchimp/mailchimp_marketing";

mailchimp.setConfig({
  apiKey: process.env.MAILCHIMP_API_KEY!,
  server: process.env.MAILCHIMP_SERVER_PREFIX!,
});

export { mailchimp };
```

------------------------------------------------------------------------

## STEP 5 : Add a member from a Route Handler

```ts
// app/api/newsletter/route.ts
import { NextResponse } from "next/server";
import { mailchimp } from "@/lib/mailchimp";

export async function POST(req: Request) {
  const { email } = (await req.json()) as { email: string };

  await mailchimp.lists.addListMember(process.env.MAILCHIMP_AUDIENCE_ID!, {
    email_address: email, // e.g. user@example.com
    status: "pending", // "pending" = double opt-in
  });

  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 6 : Double opt-in flow

```text
Visitor            Next.js API           Mailchimp
   |                   |                     |
 submit email ------>  addListMember(pending)
   |                   |                     | sends confirm email
   | <----------------- confirm link --------|
 click confirm --------------------------->  status: subscribed
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep every Mailchimp secret server-only, never in NEXT_PUBLIC vars
✓ Use status "pending" for double opt-in and cleaner deliverability
✓ Hash-map errors: treat "already a member" as a soft success
✓ Do not send transactional mail through a marketing audience
✓ Store the server prefix; the SDK needs it on every call
✓ Segment with merge fields and tags rather than many audiences
✓ Pick Mailchimp when a visual builder matters more than an API
```
