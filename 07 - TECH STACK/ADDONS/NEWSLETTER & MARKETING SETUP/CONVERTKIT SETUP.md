# CONVERTKIT SETUP [ NEWSLETTER & MARKETING ]
------------------------------------------------------------------------

ConvertKit (Kit) is a creator-focused marketing ESP built around forms,
tags, and automation sequences. Choose it for audience-driven content
businesses. This guide wires subscribers into Next.js App Router.

NOTE: this is MARKETING email (newsletters, sequences), DISTINCT from
transactional email (receipts, password resets) covered in EMAIL SETUP.
Never route account-critical mail through a marketing sequence, and
keep the two providers and keys fully separated.

------------------------------------------------------------------------

## STEP 1 : Collect credentials

1. Sign in at https://kit.com (ConvertKit).
2. Settings -> Advanced -> API: copy the `API Secret` (v4) or key.
3. Create a Form and note its numeric `Form ID`.

------------------------------------------------------------------------

## STEP 2 : Store configuration (server-only)

```bash
# .env.local
CONVERTKIT_API_KEY="ck-server-side-secret"
CONVERTKIT_FORM_ID="1234567"
```

Secrets stay server-only. Windows: use VS Code (UTF-8) for `.env.local`,
not Notepad, to avoid a BOM breaking the parser.

------------------------------------------------------------------------

## STEP 3 : No SDK needed

ConvertKit's REST API is simple; call it with the built-in `fetch`.
Nothing to install beyond your Next.js app.

```bash
# already have everything; just confirm the toolchain
pnpm install
```

------------------------------------------------------------------------

## STEP 4 : Subscribe via a Route Handler

```ts
// app/api/newsletter/route.ts
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const { email } = (await req.json()) as { email: string };
  const formId = process.env.CONVERTKIT_FORM_ID!;

  const res = await fetch(
    `https://api.convertkit.com/v3/forms/${formId}/subscribe`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        api_key: process.env.CONVERTKIT_API_KEY,
        email, // e.g. user@example.com
      }),
    },
  );

  if (!res.ok) {
    return NextResponse.json({ ok: false }, { status: 502 });
  }
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Tag for automation

```ts
await fetch(`https://api.convertkit.com/v3/tags/${tagId}/subscribe`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ api_key: process.env.CONVERTKIT_API_KEY, email }),
});
```

------------------------------------------------------------------------

## STEP 6 : Sequence flow

```text
Visitor        Next.js API         ConvertKit
   |               |                   |
 submit --------> subscribe(form) ----> add to form
   |               tag(subscriber) ---> trigger sequence
   |                                    drip emails over days
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep CONVERTKIT_API_KEY server-only; call the API from Route Handlers
✓ Use fetch on the server; no client-side subscribe requests
✓ Drive drips with tags + sequences, not ad-hoc broadcasts
✓ Enable double opt-in in form settings for deliverability
✓ Handle non-2xx responses; surface a clean error to the UI
✓ Keep marketing (Kit) and transactional email fully separated
✓ Pick ConvertKit for creator audiences and tag-based automation
```
