# HCAPTCHA SETUP [ BOT PROTECTION ]

------------------------------------------------------------------------

hCaptcha is a privacy-focused CAPTCHA that returns a response token on
solve. You render the widget client-side and verify the token against
hCaptcha's `siteverify` endpoint on the server.

------------------------------------------------------------------------

## STEP 1 : Get Site & Secret Keys

Create an account at hCaptcha, add a site, and copy the **Site Key** and
**Secret Key**.

```bash
pnpm add @hcaptcha/react-hcaptcha
```

```text
.env.local
NEXT_PUBLIC_HCAPTCHA_SITE_KEY=10000000-ffff-ffff-ffff-000000000001
HCAPTCHA_SECRET_KEY=0x0000000000000000000000000000000000000000
```

------------------------------------------------------------------------

## STEP 2 : Render The Widget (Client)

```tsx
"use client";
import HCaptcha from "@hcaptcha/react-hcaptcha";
import { useState } from "react";

export function CommentForm() {
  const [token, setToken] = useState("");

  async function submit() {
    await fetch("/api/comment", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ token, author: "user@example.com" }),
    });
  }

  return (
    <div>
      <HCaptcha
        sitekey={process.env.NEXT_PUBLIC_HCAPTCHA_SITE_KEY!}
        onVerify={setToken}
        onExpire={() => setToken("")}
      />
      <button disabled={!token} onClick={submit}>Post</button>
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Verify Token (Server)

```ts
// app/api/comment/route.ts
import { NextRequest, NextResponse } from "next/server";

const VERIFY = "https://api.hcaptcha.com/siteverify";

export async function POST(req: NextRequest) {
  const { token } = await req.json();
  const body = new URLSearchParams({
    secret: process.env.HCAPTCHA_SECRET_KEY!,
    response: token,
  });

  const res = await fetch(VERIFY, {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body,
  });

  const data = (await res.json()) as {
    success: boolean;
    "error-codes"?: string[];
  };

  if (!data.success) {
    return NextResponse.json(
      { error: "captcha failed", codes: data["error-codes"] },
      { status: 403 },
    );
  }
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 4 : Verification Flow

```text
  User solves widget
        |
        v  response token
  Client ----> POST /api/comment
                    |
                    v
              siteverify (secret + token)
                    |
        success:true  ->  accept
        success:false ->  403 + error-codes
```

------------------------------------------------------------------------

## STEP 5 : Windows Notes

No OS-specific behaviour. Run `pnpm dev` in any Windows shell. hCaptcha
ships test keys (the ones above) that always pass — use them locally and
swap to real keys via environment variables in production. Ensure no
stray CRLF characters end up in the secret value.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always verify server-side; the client token alone proves nothing
✓ Reset token state on onExpire and onError
✓ Keep the Secret Key server-only, never in NEXT_PUBLIC_*
✓ Inspect error-codes to distinguish expired vs invalid tokens
✓ Tokens are single-use — verify once then discard
✓ Use hCaptcha test keys in CI so pipelines stay deterministic
✓ Layer with rate limiting; CAPTCHA is one control, not the only one
```
