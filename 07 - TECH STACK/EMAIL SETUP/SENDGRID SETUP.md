# SENDGRID SETUP [ SCALE + MARKETING EMAIL ]
------------------------------------------------------------------------

SendGrid (Twilio) is a high-volume email platform built for scale. It
pairs a simple transactional API with dynamic templates, marketing
campaigns, and detailed analytics. Choose it when you send at large
volume or need marketing tooling alongside transactional mail.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add @sendgrid/mail
```

`@sendgrid/mail` is the official Node.js SDK. It runs on any server
runtime in your Next.js App Router project.

------------------------------------------------------------------------

## STEP 2 : Environment Variables

Create a scoped API key in the SendGrid dashboard (Settings → API Keys)
with Mail Send permission only. Store it server-side.

```bash
# .env.local
SENDGRID_API_KEY="SG.xxxxxxxxxxxxxxxxxxxxxx"
EMAIL_FROM="noreply@example.com"
```

------------------------------------------------------------------------

## STEP 3 : Create the SendGrid Client

The SDK is configured once via a module-level side effect. Wrap it in a
server-only file so the key never reaches the browser bundle.

```ts
// lib/sendgrid.ts
import "server-only";
import sgMail from "@sendgrid/mail";

if (!process.env.SENDGRID_API_KEY) {
  throw new Error("Missing SENDGRID_API_KEY");
}

sgMail.setApiKey(process.env.SENDGRID_API_KEY);

export { sgMail };
```

------------------------------------------------------------------------

## STEP 4 : Send a Plain Email

Send only from a Server Action, Route Handler, or backend job.

```ts
// app/actions/send-notice.ts
"use server";
import { sgMail } from "@/lib/sendgrid";

export async function sendNotice(to: string, body: string) {
  const [res] = await sgMail.send({
    from: process.env.EMAIL_FROM!,
    to,
    subject: "Account notice",
    text: body,
    html: `<p>${body}</p>`,
  });

  if (res.statusCode >= 400) throw new Error("SendGrid send failed");
  return res.headers["x-message-id"];
}
```

------------------------------------------------------------------------

## STEP 5 : Dynamic Templates

Design a versioned template in the dashboard using Handlebars, then send
by template ID with a `dynamicTemplateData` payload. SendGrid renders it.

```ts
// app/actions/send-welcome.ts
"use server";
import { sgMail } from "@/lib/sendgrid";

export async function sendWelcome(to: string, name: string) {
  const [res] = await sgMail.send({
    from: process.env.EMAIL_FROM!,
    to,
    templateId: "d-xxxxxxxxxxxxxxxxxxxxxxxx",
    dynamicTemplateData: {
      name,
      confirm_url: "https://example.com/confirm",
    },
  });

  if (res.statusCode >= 400) throw new Error("SendGrid send failed");
  return res.headers["x-message-id"];
}
```

------------------------------------------------------------------------

## STEP 6 : Verify Your Sending Domain

Authenticate the domain (Settings → Sender Authentication) and publish
the CNAME records SendGrid generates. This enables branded DKIM/SPF.

```text
  ┌──────────────┐   3x CNAME (DKIM/SPF)   ┌──────────────┐
  │  Your Domain │ ──────────────────────▶ │   SendGrid   │
  └──────────────┘                         └──────────────┘
   CNAME  s1._domainkey  → provided value
   CNAME  s2._domainkey  → provided value
   CNAME  em1234         → provided value  (link/return-path)
   TXT    DMARC          → v=DMARC1; p=none; rua=mailto:dmarc@example.com
```

------------------------------------------------------------------------

## WHEN TO CHOOSE SENDGRID

```text
✓ You send at large scale (millions/month) and need throughput
✓ You need marketing campaigns and transactional in one platform
✓ You want dynamic Handlebars templates and rich analytics
✗ You want the simplest, highest-deliverability transactional-only
   provider — consider Postmark instead
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Send only from server code; scope the API key to Mail Send only
✓ Authenticate the domain and publish DKIM, SPF, and DMARC records
✓ Use dynamic templates so content changes need no redeploy
✓ Add a visible unsubscribe link and honor suppression groups
✓ Provide both text and html parts for every message
✓ Check statusCode on the response; never assume delivery
✓ Warm up dedicated IPs gradually when sending at high volume
✓ Send from noreply@example.com on an authenticated subdomain
```
