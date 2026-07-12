# POSTMARK SETUP [ TRANSACTIONAL DELIVERABILITY ]
------------------------------------------------------------------------

Postmark is a transactional-first provider prized for fast, reliable
inbox placement. It deliberately separates transactional and bulk mail
into distinct message streams so reputation stays clean. Choose Postmark
when delivery speed and deliverability matter more than marketing tools.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add postmark
```

The official `postmark` SDK wraps the HTTP API and is safe to run from
any Node.js server runtime in your Next.js App Router project.

------------------------------------------------------------------------

## STEP 2 : Environment Variables

Create a Server Token in the Postmark dashboard (Servers → API Tokens).
Store it server-side only — never with a `NEXT_PUBLIC_` prefix.

```bash
# .env.local
POSTMARK_SERVER_TOKEN="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
EMAIL_FROM="noreply@example.com"
```

------------------------------------------------------------------------

## STEP 3 : Create the Postmark Client

```ts
// lib/postmark.ts
import "server-only";
import { ServerClient } from "postmark";

if (!process.env.POSTMARK_SERVER_TOKEN) {
  throw new Error("Missing POSTMARK_SERVER_TOKEN");
}

export const postmark = new ServerClient(
  process.env.POSTMARK_SERVER_TOKEN
);
```

------------------------------------------------------------------------

## STEP 4 : Message Streams

Postmark routes every message through a stream. Keep transactional and
broadcast mail apart so a marketing blast never taints receipt delivery.

```text
  ┌─────────────────────────────────────────────┐
  │                POSTMARK SERVER               │
  ├──────────────────────┬──────────────────────┤
  │  transactional       │  broadcast           │
  │  (receipts, resets)  │  (newsletters)       │
  │  stream: "outbound"  │  stream: "broadcast" │
  └──────────────────────┴──────────────────────┘
   Broadcast streams REQUIRE an unsubscribe link.
```

------------------------------------------------------------------------

## STEP 5 : Send a Plain Transactional Email

Send only from a Server Action, Route Handler, or backend job.

```ts
// app/actions/send-receipt.ts
"use server";
import { postmark } from "@/lib/postmark";

export async function sendReceipt(to: string, orderId: string) {
  const res = await postmark.sendEmail({
    From: process.env.EMAIL_FROM!,
    To: to,
    Subject: `Receipt for order ${orderId}`,
    HtmlBody: `<p>Thanks! Your order ${orderId} is confirmed.</p>`,
    TextBody: `Thanks! Your order ${orderId} is confirmed.`,
    MessageStream: "outbound",
  });

  if (res.ErrorCode !== 0) throw new Error(res.Message);
  return res.MessageID;
}
```

------------------------------------------------------------------------

## STEP 6 : Send With a Template

Design templates in the dashboard, then send by alias with a variable
model. Postmark renders server-side and tracks each template's stats.

```ts
// app/actions/send-reset.ts
"use server";
import { postmark } from "@/lib/postmark";

export async function sendReset(to: string, url: string) {
  const res = await postmark.sendEmailWithTemplate({
    From: process.env.EMAIL_FROM!,
    To: to,
    TemplateAlias: "password-reset",
    TemplateModel: { action_url: url, product_name: "Acme" },
    MessageStream: "outbound",
  });

  if (res.ErrorCode !== 0) throw new Error(res.Message);
  return res.MessageID;
}
```

------------------------------------------------------------------------

## STEP 7 : Verify Your Sending Domain

Add a Sender Signature or domain, then publish the DNS records so mail
authenticates. DKIM and Return-Path are required for good placement.

```text
  DKIM     → CNAME  yyyy._domainkey → provided value
  SPF      → TXT    v=spf1 include:spf.mtasv.net ~all
  Return-Path (CNAME) → pm-bounces → provided value
  DMARC    → TXT    v=DMARC1; p=none; rua=mailto:dmarc@example.com
```

------------------------------------------------------------------------

## WHEN TO CHOOSE POSTMARK

```text
✓ You need the fastest, most reliable transactional delivery
✓ You want clean reputation via separate message streams
✓ Your volume is receipts, resets, alerts — not marketing blasts
✗ You want a full marketing suite — use SendGrid instead
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Send only from server code; keep the Server Token off the client
✓ Verify the domain and publish DKIM, SPF, Return-Path, and DMARC
✓ Route transactional and broadcast mail through separate streams
✓ Always include a TextBody alongside HtmlBody for accessibility
✓ Add a visible unsubscribe link to every broadcast-stream email
✓ Check ErrorCode on every response; never assume delivery
✓ Use a noreply@example.com sender on an authenticated domain
```
