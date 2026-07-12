# GOOGLE ANALYTICS SETUP [ GA4 WEB ANALYTICS ]
------------------------------------------------------------------------
## STEP 1 : Install @next/third-parties

The official `@next/third-parties` package loads GA4 efficiently and
off the main thread, without hand-writing script tags.

```bash
pnpm add @next/third-parties
```

Create a GA4 property in the Google Analytics console and copy the
Measurement ID (looks like `G-XXXXXXXXXX`).

```bash
# .env.local
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

------------------------------------------------------------------------
## STEP 2 : Add GoogleAnalytics to the Layout

Render the component at the end of `<body>` in the root layout.

```tsx
// app/layout.tsx
import { GoogleAnalytics } from "@next/third-parties/google";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <GoogleAnalytics gaId={process.env.NEXT_PUBLIC_GA_ID!} />
      </body>
    </html>
  );
}
```

The component handles page-view tracking across App Router navigation.

------------------------------------------------------------------------
## STEP 3 : Send Custom Events

Use the `sendGAEvent` helper from client components.

```tsx
"use client";
import { sendGAEvent } from "@next/third-parties/google";

export function BuyButton({ sku }: { sku: string }) {
  return (
    <button
      onClick={() =>
        sendGAEvent("event", "purchase_click", { sku }) // no PII
      }
    >
      Buy
    </button>
  );
}
```

------------------------------------------------------------------------
## STEP 4 : How the Data Flows

```text
  visitor --> GoogleAnalytics (gtag.js) --> GA4 property
                    |                          |
             page_view auto              custom events
                                              |
                            reports / explorations / audiences
```

------------------------------------------------------------------------
## STEP 5 : Consent Mode (Important)

GA4 uses cookies, so in the EU/UK you must gate it behind consent. Set
defaults to denied and update after the user accepts.

```tsx
"use client";
// Call before GA loads (e.g. in a consent-banner handler)
declare global {
  interface Window {
    gtag: (...args: unknown[]) => void;
  }
}

export function grantAnalyticsConsent() {
  window.gtag?.("consent", "update", {
    analytics_storage: "granted",
  });
}
```

```text
  default: analytics_storage = "denied"
  user accepts banner --> consent update "granted" --> GA collects
```

You must show a cookie banner for GA4 in cookie-regulated regions.

------------------------------------------------------------------------
## STEP 6 : Keep It Privacy-Safe

Never pass emails, names, or user ids into event params — GA4 forbids
sending PII and it can get your property suspended.

```tsx
// Good: category + non-personal id
sendGAEvent("event", "add_to_cart", { sku: "TSHIRT-01", value: 25 });

// Bad: PII in analytics
sendGAEvent("event", "signup", { email: "user@example.com" });
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Load GA4 via @next/third-parties, not raw script tags
✓ Keep the Measurement ID in an env var
✓ Never send PII (email, name, user id) in event params
✓ Default consent to denied; grant only after acceptance
✓ Show a cookie banner in the EU/UK — GA4 uses cookies
✓ Name events with GA4 recommended conventions where possible
✓ Track meaningful conversions, not every click
✓ Verify events in GA4 DebugView before trusting reports
```
