# NODEMAILER SETUP [ SMTP / SELF-HOSTED SENDING ]
------------------------------------------------------------------------

Nodemailer is the classic Node.js library for sending mail over SMTP. It
talks to any SMTP server — your own Postfix, a corporate relay, or a
provider's SMTP endpoint. Choose it when you must use your own SMTP
infrastructure rather than a vendor HTTP API.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add nodemailer
pnpm add -D @types/nodemailer
```

`nodemailer` is the runtime library; the types package gives full
TypeScript support in your Next.js App Router project.

------------------------------------------------------------------------

## STEP 2 : Environment Variables

Store SMTP credentials server-side only. Never prefix with
`NEXT_PUBLIC_`.

```bash
# .env.local
SMTP_HOST="smtp.example.com"
SMTP_PORT="587"
SMTP_USER="postmaster@example.com"
SMTP_PASS="super-secret-password"
EMAIL_FROM="noreply@example.com"
```

Port 587 uses STARTTLS; port 465 uses implicit TLS (`secure: true`).

------------------------------------------------------------------------

## STEP 3 : Create the SMTP Transport

Reuse a single transporter so the connection pool is shared across
requests. Keep it in a server-only module.

```ts
// lib/mailer.ts
import "server-only";
import nodemailer from "nodemailer";

export const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: Number(process.env.SMTP_PORT),
  secure: Number(process.env.SMTP_PORT) === 465,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
  pool: true,
});
```

------------------------------------------------------------------------

## STEP 4 : Send From the Server

Send only from a Server Action, Route Handler, or backend job — SMTP
credentials must never reach the client.

```ts
// app/actions/send-mail.ts
"use server";
import { transporter } from "@/lib/mailer";

export async function sendMail(to: string, subject: string, html: string) {
  const info = await transporter.sendMail({
    from: process.env.EMAIL_FROM!,
    to,
    subject,
    html,
    text: html.replace(/<[^>]+>/g, " "),
  });

  return info.messageId;
}
```

------------------------------------------------------------------------

## STEP 5 : Verify the Connection

Call `verify()` at startup or in a health check to fail fast on bad
credentials instead of discovering them mid-send.

```ts
// scripts/verify-smtp.ts
import { transporter } from "@/lib/mailer";

const ok = await transporter.verify();
console.log(ok ? "SMTP ready" : "SMTP not ready");
```

------------------------------------------------------------------------

## STEP 6 : Serverless Warning

```text
  ┌────────────────────────────────────────────────────┐
  │  SERVERLESS COLD START + SMTP = SLOW / FLAKY        │
  ├────────────────────────────────────────────────────┤
  │  request ─▶ cold start ─▶ open SMTP socket ─▶ TLS   │
  │            ─▶ AUTH ─▶ send ─▶ tear down             │
  │                                                     │
  │  Each cold invocation re-establishes the handshake. │
  │  Connection pools do not survive between invokes.   │
  └────────────────────────────────────────────────────┘
```

SMTP is connection-oriented and stateful. On serverless (Vercel, Lambda)
cold starts add latency, pools cannot be reused, and long-lived sockets
may be killed. For serverless, prefer an HTTP API provider (Resend,
Postmark, SendGrid). Use Nodemailer on a long-running server or worker.

------------------------------------------------------------------------

## STEP 7 : Verify Your Sending Domain

Even with your own SMTP, publish authentication records or mail lands in
spam. These live on the domain you send `EMAIL_FROM` from.

```text
  SPF   → TXT  v=spf1 ip4:203.0.113.10 include:example.com ~all
  DKIM  → TXT  selector._domainkey → your public key
  DMARC → TXT  v=DMARC1; p=none; rua=mailto:dmarc@example.com
  PTR   → reverse DNS for the sending IP must resolve
```

------------------------------------------------------------------------

## WHEN TO CHOOSE NODEMAILER

```text
✓ You must send through your own or a corporate SMTP relay
✓ You self-host mail and want no third-party API dependency
✓ You run a long-lived Node server or background worker
✗ You deploy to serverless — cold starts make SMTP slow/flaky
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Send only from server code; keep SMTP credentials off the client
✓ Verify the sending domain with SPF, DKIM, DMARC, and reverse DNS
✓ Reuse a pooled transporter instead of creating one per request
✓ Call verify() at startup to fail fast on bad credentials
✓ Add a visible unsubscribe link to any non-transactional email
✓ Always include a text part alongside html for accessibility
✓ Avoid Nodemailer on serverless; use an HTTP provider there
✓ Send from noreply@example.com on an authenticated domain
```
