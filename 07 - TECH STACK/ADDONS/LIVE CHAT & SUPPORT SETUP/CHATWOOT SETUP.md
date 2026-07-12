# CHATWOOT SETUP [ LIVE CHAT & SUPPORT ]
------------------------------------------------------------------------

Chatwoot is an open-source, self-hostable customer support suite: live
chat, shared inbox, and multi-channel routing. Choose it when you want
data ownership and no per-seat SaaS fees. This guide covers self-hosting
plus embedding the website widget in Next.js App Router.

------------------------------------------------------------------------

## STEP 1 : Self-host with Docker

Chatwoot ships an official Compose stack (Rails app + Postgres + Redis).

```bash
git clone https://github.com/chatwoot/chatwoot.git
cd chatwoot
cp .env.example .env
docker compose up -d
```

Open http://localhost:3000 and create the admin account. Windows: run
Docker Desktop with the WSL2 backend; clone into the WSL filesystem
(not `/mnt/c`) for far better volume I/O.

------------------------------------------------------------------------

## STEP 2 : Create a website inbox

1. Sign in -> Inboxes -> Add Inbox -> Website.
2. Set the channel name and your site domain.
3. Copy the `Website Token` from the installation snippet.

------------------------------------------------------------------------

## STEP 3 : Store configuration

```bash
# .env.local
NEXT_PUBLIC_CHATWOOT_TOKEN="your-website-token"
NEXT_PUBLIC_CHATWOOT_BASE_URL="https://chat.yourdomain.com"
```

Both values are public (they appear in the client snippet). Point the
base URL at your deployed instance, not localhost, in production.

------------------------------------------------------------------------

## STEP 4 : Load the SDK script

Chatwoot has no npm package; load its SDK with `next/script`.

```tsx
// components/chatwoot.tsx
"use client";

import Script from "next/script";

export function Chatwoot() {
  const base = process.env.NEXT_PUBLIC_CHATWOOT_BASE_URL!;
  const token = process.env.NEXT_PUBLIC_CHATWOOT_TOKEN!;

  return (
    <Script
      id="chatwoot"
      strategy="lazyOnload"
      src={`${base}/packs/js/sdk.js`}
      onLoad={() => {
        // @ts-expect-error injected global
        window.chatwootSDK.run({ websiteToken: token, baseUrl: base });
      }}
    />
  );
}
```

Mount `<Chatwoot />` in `app/layout.tsx` so it runs on every route.

------------------------------------------------------------------------

## STEP 5 : Identify users securely

Set an HMAC identifier hash server-side (Settings -> Inbox -> enforce
identity validation), then push it to the widget.

```ts
// after window.$chatwoot is ready
window.$chatwoot.setUser("USER_ID", {
  name: "Ada",
  email: "user@example.com",
  identifier_hash: serverComputedHash,
});
```

------------------------------------------------------------------------

## STEP 6 : Topology

```text
Next.js frontend        Your VPS (Docker)
      |                  +----------------------+
  sdk.js -------------->  Rails  Postgres  Redis
  setUser(hash) ------->  agent dashboard
                          +----------------------+
                          you own all the data
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Self-host behind HTTPS with a reverse proxy (Caddy or Nginx)
✓ Enable identity validation and compute identifier_hash server-side
✓ Back up Postgres regularly; it holds every conversation
✓ Load sdk.js with strategy="lazyOnload" to protect page speed
✓ Pin the Chatwoot image tag; test upgrades on staging first
✓ On Windows, develop under WSL2, not /mnt/c, for volume speed
✓ Prefer Chatwoot when data ownership beats managed convenience
```
