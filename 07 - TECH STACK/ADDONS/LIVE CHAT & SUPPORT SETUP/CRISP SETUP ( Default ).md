# CRISP SETUP [ LIVE CHAT & SUPPORT ]
------------------------------------------------------------------------

Crisp is the default live chat widget for this stack: a lightweight,
privacy-friendly inbox with a generous free tier, a single-script embed,
and a clean JavaScript SDK for identity + events. This guide wires Crisp
into a Next.js App Router app the right way.

------------------------------------------------------------------------

## STEP 1 : Create the workspace

1. Sign up at https://app.crisp.chat and create a website.
2. Open Settings -> Website Settings -> Setup instructions.
3. Copy your `Website ID` (a UUID). This is public and safe to ship.

------------------------------------------------------------------------

## STEP 2 : Store the Website ID

Only the Website ID is needed on the client, so prefix it `NEXT_PUBLIC_`.

```bash
# .env.local
NEXT_PUBLIC_CRISP_WEBSITE_ID="00000000-0000-0000-0000-000000000000"
```

Windows note: use `.env.local` (no BOM). VS Code saves UTF-8 by default;
avoid Notepad, which can inject a BOM that breaks parsing.

------------------------------------------------------------------------

## STEP 3 : Install the SDK

```bash
pnpm add crisp-sdk-web
```

------------------------------------------------------------------------

## STEP 4 : Mount the widget

Create a client component that boots Crisp once after hydration.

```tsx
// components/crisp-chat.tsx
"use client";

import { useEffect } from "react";
import { Crisp } from "crisp-sdk-web";

export function CrispChat() {
  useEffect(() => {
    const id = process.env.NEXT_PUBLIC_CRISP_WEBSITE_ID;
    if (!id) return;
    Crisp.configure(id);
  }, []);

  return null;
}
```

Render it in the root layout so it loads on every route.

```tsx
// app/layout.tsx
import { CrispChat } from "@/components/crisp-chat";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <CrispChat />
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Identify signed-in users

Attach identity so agents see who they are talking to.

```ts
// lib/crisp-identify.ts
import { Crisp } from "crisp-sdk-web";

export function identify(user: { email: string; name: string }) {
  Crisp.user.setEmail(user.email);      // e.g. user@example.com
  Crisp.user.setNickname(user.name);
  Crisp.session.setData({ plan: "pro" });
}
```

------------------------------------------------------------------------

## STEP 6 : Data flow

```text
Browser                 Next.js                Crisp Cloud
   |                       |                        |
   | load layout --------> | render <CrispChat/>    |
   | configure(WebsiteID)--------------------------> boot widget
   | identify(email,name)--------------------------> attach session
   | message ------------------------------------->  agent inbox
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Ship only the public Website ID to the client, never secrets
✓ Boot Crisp inside useEffect so it never runs during SSR
✓ Load the widget in the root layout, mount it exactly once
✓ Identify users post-auth for richer support conversations
✓ Use session.setData for plan / tenant, not sensitive PII
✓ Lazy-load on low-priority routes to protect Core Web Vitals
✓ Keep Crisp as the default; swap only if you outgrow the free tier
```
