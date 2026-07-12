# RESEND SETUP [ TRANSACTIONAL + REACT EMAIL ]
------------------------------------------------------------------------

Resend is a developer-first email API built around React Email. It is the
fastest path from a Next.js App Router project to verified, deliverable
transactional mail. All sending happens server-side with a secret API key.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add resend
pnpm add react-email @react-email/components -D
```

`resend` is the runtime SDK. `react-email` gives you the local preview
server and JSX components for building templates.

------------------------------------------------------------------------

## STEP 2 : Environment Variables

Create the key in the Resend dashboard, then store it server-side only.
Never expose it with a `NEXT_PUBLIC_` prefix.

```bash
# .env.local
RESEND_API_KEY="re_xxxxxxxxxxxxxxxxxxxxxxxx"
EMAIL_FROM="noreply@example.com"
```

------------------------------------------------------------------------

## STEP 3 : Create the Resend Client

Keep a single shared client in a server-only module.

```ts
// lib/resend.ts
import "server-only";
import { Resend } from "resend";

if (!process.env.RESEND_API_KEY) {
  throw new Error("Missing RESEND_API_KEY");
}

export const resend = new Resend(process.env.RESEND_API_KEY);
```

------------------------------------------------------------------------

## STEP 4 : Build a React Email Template

Templates are plain React components. They render to HTML on the server.

```tsx
// emails/welcome.tsx
import {
  Html, Head, Body, Container, Heading, Text, Button,
} from "@react-email/components";

interface WelcomeEmailProps {
  name: string;
  url: string;
}

export default function WelcomeEmail({ name, url }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: "sans-serif", background: "#f6f6f6" }}>
        <Container style={{ padding: "24px" }}>
          <Heading>Welcome, {name}</Heading>
          <Text>Thanks for signing up. Confirm your account below.</Text>
          <Button href={url} style={{ background: "#000", color: "#fff", padding: "12px 20px" }}>
            Confirm account
          </Button>
        </Container>
      </Body>
    </Html>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Send From the Server

Send only from a Server Action, Route Handler, or backend job — never
from a client component, or your API key leaks.

```ts
// app/actions/send-welcome.ts
"use server";
import { resend } from "@/lib/resend";
import WelcomeEmail from "@/emails/welcome";

export async function sendWelcome(to: string, name: string) {
  const { data, error } = await resend.emails.send({
    from: process.env.EMAIL_FROM!,
    to,
    subject: "Welcome aboard",
    react: WelcomeEmail({ name, url: "https://example.com/confirm" }),
  });

  if (error) throw new Error(error.message);
  return data?.id;
}
```

------------------------------------------------------------------------

## STEP 6 : Local Preview

Run the React Email dev server to iterate on templates in the browser
with hot reload, no real sends.

```bash
pnpm email dev
# opens http://localhost:3000 with a live preview of /emails
```

------------------------------------------------------------------------

## STEP 7 : Verify Your Sending Domain

Add the domain in the Resend dashboard and publish the DNS records it
provides. Verification unlocks deliverability and removes the sandbox.

```text
  ┌──────────────┐    DNS records     ┌──────────────┐
  │  Your Domain │ ─────────────────▶ │    Resend    │
  └──────────────┘                    └──────────────┘
   TXT  (SPF)   → v=spf1 include:amazonses.com ~all
   CNAME (DKIM) → resend._domainkey → provided value
   TXT  (DMARC) → v=DMARC1; p=none; rua=mailto:dmarc@example.com
```

Propagation can take up to 48h; most records resolve within minutes.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Send only from server code; never ship the API key to the client
✓ Verify the sending domain and publish SPF, DKIM, and DMARC records
✓ Use a noreply@example.com from-address on a subdomain like mail.
✓ Build and preview templates with `pnpm email dev` before shipping
✓ Add a visible unsubscribe link to any non-transactional email
✓ Handle the { data, error } result; never assume a send succeeded
✓ Set a DMARC policy and monitor reports to protect your domain
✓ Keep idempotency in mind — retry sends with a stable key
```
