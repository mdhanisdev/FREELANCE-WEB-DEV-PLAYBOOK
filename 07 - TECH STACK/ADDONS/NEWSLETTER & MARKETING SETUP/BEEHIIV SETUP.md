# BEEHIIV SETUP [ NEWSLETTER & MARKETING ]
------------------------------------------------------------------------

beehiiv is a modern newsletter platform with growth tooling, referrals,
and monetization built in. Choose it when the newsletter itself is the
product. This guide wires subscriptions into Next.js App Router via the
beehiiv v2 API.

NOTE: this is MARKETING email (newsletter issues, broadcasts), DISTINCT
from transactional email (receipts, password resets) in EMAIL SETUP. A
marketing unsubscribe must never block a transactional account email,
so keep the providers and API keys separate.

------------------------------------------------------------------------

## STEP 1 : Collect credentials

1. Sign in at https://www.beehiiv.com.
2. Settings -> Integrations -> API: create a v2 API key.
3. Copy your `Publication ID` (starts with `pub_`).

------------------------------------------------------------------------

## STEP 2 : Store configuration (server-only)

```bash
# .env.local
BEEHIIV_API_KEY="server-side-secret"
BEEHIIV_PUBLICATION_ID="pub_00000000-0000-0000-0000-000000000000"
```

Both are secret. Windows: edit `.env.local` in VS Code (UTF-8), never
Notepad, so no BOM sneaks in.

------------------------------------------------------------------------

## STEP 3 : No SDK required

The v2 REST API is Bearer-token based; use the built-in `fetch`.

```bash
pnpm install
```

------------------------------------------------------------------------

## STEP 4 : Subscribe via a Route Handler

```ts
// app/api/newsletter/route.ts
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const { email } = (await req.json()) as { email: string };
  const pub = process.env.BEEHIIV_PUBLICATION_ID!;

  const res = await fetch(
    `https://api.beehiiv.com/v2/publications/${pub}/subscriptions`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${process.env.BEEHIIV_API_KEY}`,
      },
      body: JSON.stringify({
        email, // e.g. user@example.com
        reactivate_existing: false,
        utm_source: "website",
      }),
    },
  );

  if (!res.ok) return NextResponse.json({ ok: false }, { status: 502 });
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Attach acquisition data

Pass UTM and referral fields so beehiiv attributes growth correctly.

```ts
const body = {
  email,
  utm_source: "product",
  utm_medium: "in_app",
  referring_site: "app.yourdomain.com",
};
```

------------------------------------------------------------------------

## STEP 6 : Growth flow

```text
Visitor        Next.js API             beehiiv
   |               |                      |
 submit --------> POST /subscriptions --> subscriber + UTM
   |               (Bearer key)           referral / growth tools
   |                                      newsletter issue send
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep BEEHIIV_API_KEY server-only; send it only as a Bearer header
✓ Call the v2 API from Route Handlers, never from the browser
✓ Pass UTM + referral fields to power beehiiv growth analytics
✓ Set reactivate_existing deliberately to respect prior opt-outs
✓ Validate email and handle non-2xx responses gracefully
✓ Keep newsletter (beehiiv) and transactional email fully separate
✓ Pick beehiiv when the newsletter is the core product
```
