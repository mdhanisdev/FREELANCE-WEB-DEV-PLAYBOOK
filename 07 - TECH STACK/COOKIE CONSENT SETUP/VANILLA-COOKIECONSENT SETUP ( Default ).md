# VANILLA-COOKIECONSENT SETUP [ COOKIE CONSENT ]
------------------------------------------------------------------------
vanilla-cookieconsent is our default GDPR/ePrivacy consent manager: no
framework lock-in, category-based blocking, and a small footprint. This
guide integrates it into Next.js App Router with TypeScript and pnpm,
gating analytics until the user opts in.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add vanilla-cookieconsent
```
------------------------------------------------------------------------
## STEP 2 : Define the Consent Config

```ts
// lib/consent-config.ts
import type { CookieConsentConfig } from "vanilla-cookieconsent";

export const config: CookieConsentConfig = {
  categories: {
    necessary: { enabled: true, readOnly: true },
    analytics: {},
  },
  guiOptions: {
    consentModal: { layout: "box", position: "bottom right" },
  },
  language: {
    default: "en",
    translations: {
      en: {
        consentModal: {
          title: "We use cookies",
          description: "Choose which cookies to allow.",
          acceptAllBtn: "Accept all",
          acceptNecessaryBtn: "Reject all",
          showPreferencesBtn: "Manage preferences",
        },
        preferencesModal: {
          title: "Preferences",
          acceptAllBtn: "Accept all",
          acceptNecessaryBtn: "Reject all",
          savePreferencesBtn: "Save",
          sections: [
            { title: "Necessary", linkedCategory: "necessary" },
            { title: "Analytics", linkedCategory: "analytics" },
          ],
        },
      },
    },
  },
};
```
------------------------------------------------------------------------
## STEP 3 : Build a Client Initializer

```tsx
// components/cookie-consent.tsx
"use client";

import { useEffect } from "react";
import * as CookieConsent from "vanilla-cookieconsent";
import "vanilla-cookieconsent/dist/cookieconsent.css";
import { config } from "@/lib/consent-config";

export function CookieConsentBanner() {
  useEffect(() => {
    CookieConsent.run(config);
  }, []);
  return null;
}
```

```text
Consent flow
  page load ──▶ banner ──▶ user choice ──▶ acceptedCategory('analytics')?
                                              │yes → load GA
                                              │no  → stay blocked
```
------------------------------------------------------------------------
## STEP 4 : Mount in the Root Layout

```tsx
// app/layout.tsx
import { CookieConsentBanner } from "@/components/cookie-consent";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <CookieConsentBanner />
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Gate Scripts on Consent

```ts
// lib/analytics.ts
import * as CookieConsent from "vanilla-cookieconsent";

export function loadAnalytics() {
  if (!CookieConsent.acceptedCategory("analytics")) return;
  const s = document.createElement("script");
  s.src = "https://www.googletagmanager.com/gtag/js?id=G-XXXX";
  s.async = true;
  document.head.appendChild(s);
}
```

Call `loadAnalytics()` inside the config `onConsent` / `onChange` hooks.
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# open http://localhost:3000 — reject, then check no GA cookie set
```

Windows note: consent is stored in a `cc_cookie` browser cookie, not on
disk, so behavior is identical across OSes. Clear it via DevTools to
re-trigger the banner during testing.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Block all non-necessary scripts until explicit opt-in
✓ Mark the necessary category readOnly and enabled
✓ Offer equally prominent Accept and Reject buttons (GDPR)
✓ Provide a "Manage preferences" re-entry point on every page
✓ Load analytics inside onConsent/onChange, never on mount
✓ Version your consent config; re-prompt when categories change
✓ Import cookieconsent.css once in the client component
✓ Log consent state changes for audit/compliance evidence
```
