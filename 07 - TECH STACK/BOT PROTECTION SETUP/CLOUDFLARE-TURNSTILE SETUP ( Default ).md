# CLOUDFLARE TURNSTILE SETUP [ BOT PROTECTION ]

------------------------------------------------------------------------

Turnstile is Cloudflare's privacy-first CAPTCHA alternative. It runs a
non-interactive challenge in the browser, hands the client a token, and
you verify that token server-side before trusting the request.

------------------------------------------------------------------------

## STEP 1 : Create Widget & Get Keys

Log in to the Cloudflare dashboard, open **Turnstile**, and add a site.
You receive a **Site Key** (public) and a **Secret Key** (server-only).

```bash
pnpm add react-turnstile
```

```text
.env.local
NEXT_PUBLIC_TURNSTILE_SITE_KEY=1x00000000000000000000AA
TURNSTILE_SECRET_KEY=2x0000000000000000000000000000000AA
```

------------------------------------------------------------------------

## STEP 2 : Render The Widget (Client)

The widget mounts client-side and stores the token in form state.

```tsx
"use client";
import { Turnstile } from "react-turnstile";
import { useState } from "react";

export function ProtectedForm() {
  const [token, setToken] = useState("");

  async function onSubmit(e: React.FormEvent) {
    e.preventDefault();
    await fetch("/api/contact", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ token, email: "user@example.com" }),
    });
  }

  return (
    <form onSubmit={onSubmit}>
      <Turnstile
        sitekey={process.env.NEXT_PUBLIC_TURNSTILE_SITE_KEY!}
        onVerify={setToken}
      />
      <button type="submit" disabled={!token}>Send</button>
    </form>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Verify Token Server-Side

Never trust the client. POST the token to the `siteverify` endpoint from
a Route Handler and reject the request on failure.

```ts
// app/api/contact/route.ts
import { NextRequest, NextResponse } from "next/server";

const VERIFY = "https://challenges.cloudflare.com/turnstile/v0/siteverify";

export async function POST(req: NextRequest) {
  const { token } = await req.json();
  const ip = req.headers.get("cf-connecting-ip") ?? "";

  const res = await fetch(VERIFY, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      secret: process.env.TURNSTILE_SECRET_KEY,
      response: token,
      remoteip: ip,
    }),
  });

  const data = (await res.json()) as { success: boolean };
  if (!data.success) {
    return NextResponse.json({ error: "bot check failed" }, { status: 403 });
  }
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 4 : Request Flow

```text
  Browser                Next.js Route          Cloudflare
  -------                -------------          ----------
  render widget
  solve challenge ---->  (token)
  submit form  ------->  POST /api/contact
                         verify token ------->  siteverify
                         success? <-----------  { success }
                         200 / 403
```

------------------------------------------------------------------------

## STEP 5 : Windows Notes

Use `pnpm dev` from PowerShell or Git Bash; both work. When copying keys
into `.env.local`, avoid trailing CRLF whitespace — some editors add it.
Turnstile itself has no OS-specific behaviour; verification is a plain
HTTPS call from the server runtime.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always verify the token server-side, never client-only
✓ Keep the Secret Key out of NEXT_PUBLIC_* variables
✓ Each token is single-use — verify exactly once
✓ Pass remoteip for stronger scoring when behind Cloudflare
✓ Use a short timeout + retry around the siteverify fetch
✓ Reset the widget after a failed submit to force a fresh token
✓ Use test keys (1x/2x/3x) in CI, real keys only in production
```
