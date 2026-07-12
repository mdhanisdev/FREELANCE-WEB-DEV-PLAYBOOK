# RECAPTCHA SETUP [ BOT PROTECTION ]

------------------------------------------------------------------------

Google reCAPTCHA v3 scores every request from 0.0 (bot) to 1.0 (human)
with no user interaction. You collect the token on the client and verify
it, plus its score, on the server before acting on the request.

------------------------------------------------------------------------

## STEP 1 : Register Site & Get Keys

In the reCAPTCHA admin console register a site as **v3**. You get a
**Site Key** and a **Secret Key**.

```bash
pnpm add react-google-recaptcha-v3
```

```text
.env.local
NEXT_PUBLIC_RECAPTCHA_SITE_KEY=6LxxxxxxxxxxxxxxxxxxxxxxxxxxPUBLIC
RECAPTCHA_SECRET_KEY=6LxxxxxxxxxxxxxxxxxxxxxxxxxxSECRET
```

------------------------------------------------------------------------

## STEP 2 : Wrap The App With The Provider

The provider injects the reCAPTCHA script once for the whole tree.

```tsx
// app/providers.tsx
"use client";
import { GoogleReCaptchaProvider } from "react-google-recaptcha-v3";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <GoogleReCaptchaProvider
      reCaptchaKey={process.env.NEXT_PUBLIC_RECAPTCHA_SITE_KEY!}
    >
      {children}
    </GoogleReCaptchaProvider>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Execute & Submit (Client)

Call `executeRecaptcha` with an action name right before submit.

```tsx
"use client";
import { useGoogleReCaptcha } from "react-google-recaptcha-v3";

export function SignupForm() {
  const { executeRecaptcha } = useGoogleReCaptcha();

  async function onSubmit() {
    if (!executeRecaptcha) return;
    const token = await executeRecaptcha("signup");
    await fetch("/api/signup", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ token, email: "user@example.com" }),
    });
  }

  return <button onClick={onSubmit}>Create account</button>;
}
```

------------------------------------------------------------------------

## STEP 4 : Verify Token & Score (Server)

```ts
// app/api/signup/route.ts
import { NextRequest, NextResponse } from "next/server";

const VERIFY = "https://www.google.com/recaptcha/api/siteverify";

export async function POST(req: NextRequest) {
  const { token } = await req.json();
  const body = new URLSearchParams({
    secret: process.env.RECAPTCHA_SECRET_KEY!,
    response: token,
  });

  const res = await fetch(VERIFY, { method: "POST", body });
  const data = (await res.json()) as {
    success: boolean;
    score: number;
    action: string;
  };

  if (!data.success || data.action !== "signup" || data.score < 0.5) {
    return NextResponse.json({ error: "blocked" }, { status: 403 });
  }
  return NextResponse.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Scoring Flow

```text
  Client                     Server
  ------                     ------
  executeRecaptcha("signup")
        |
        v  token
  POST /api/signup  ------->  siteverify
                             { success, score, action }
                                     |
                    score >= 0.5  &&  action match ?
                             yes -> 200      no -> 403
```

Tune the threshold per action; login and payment should be stricter than
a newsletter form.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` behaves identically on Windows. If a corporate proxy blocks
`google.com`, siteverify will fail locally — test on a network that
allows outbound HTTPS to Google. Watch for CRLF in `.env.local` keys.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Verify success, score, AND the expected action name
✓ Set per-action thresholds (payments stricter than signups)
✓ Keep the Secret Key server-only, never NEXT_PUBLIC_*
✓ Log low scores to tune thresholds over time
✓ Add a fallback (v2 checkbox / email link) for borderline scores
✓ Load the provider once at the root, not per form
✓ Do not treat reCAPTCHA as the only defence — pair with rate limits
```
