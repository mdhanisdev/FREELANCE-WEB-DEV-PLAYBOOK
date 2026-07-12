# TAWK SETUP [ LIVE CHAT & SUPPORT ]
------------------------------------------------------------------------

Tawk.to is a genuinely free live chat widget (no message caps, no seat
limits). It is the lowest-friction option when budget is the priority.
This guide embeds the Tawk widget in a Next.js App Router app and adds
optional visitor attributes with a secure hash.

------------------------------------------------------------------------

## STEP 1 : Create a property

1. Sign up at https://www.tawk.to.
2. Add a Property and a Chat Widget.
3. Admin -> Channels -> Chat Widget: note the `Property ID` and
   `Widget ID` from the direct-chat-link / embed snippet.

------------------------------------------------------------------------

## STEP 2 : Store the IDs

```bash
# .env.local
NEXT_PUBLIC_TAWK_PROPERTY_ID="0000000000000000000000"
NEXT_PUBLIC_TAWK_WIDGET_ID="1abcd2efg"
```

Both IDs are public and appear in the widget URL. Windows: save
`.env.local` as UTF-8 (VS Code default), never via Notepad with a BOM.

------------------------------------------------------------------------

## STEP 3 : Embed with next/script

Tawk provides no npm package; inject its loader via `next/script`.

```tsx
// components/tawk.tsx
"use client";

import Script from "next/script";

export function Tawk() {
  const property = process.env.NEXT_PUBLIC_TAWK_PROPERTY_ID!;
  const widget = process.env.NEXT_PUBLIC_TAWK_WIDGET_ID!;

  return (
    <Script id="tawk" strategy="lazyOnload">
      {`
        var Tawk_API = Tawk_API || {};
        Tawk_API.customStyle = { visibility: { desktop: { position: "br" } } };
        (function () {
          var s = document.createElement("script");
          s.async = true;
          s.src = "https://embed.tawk.to/${property}/${widget}";
          s.charset = "UTF-8";
          s.setAttribute("crossorigin", "*");
          document.body.appendChild(s);
        })();
      `}
    </Script>
  );
}
```

Render `<Tawk />` in `app/layout.tsx`.

------------------------------------------------------------------------

## STEP 4 : Set visitor attributes

Enable Secure Mode (Admin -> Property Settings) and hash the email with
your API key on the server, then pass it to the widget.

```ts
// browser, once Tawk_API is ready
window.Tawk_API.setAttributes(
  { name: "Ada", email: "user@example.com", hash: serverHash },
  (err: unknown) => err && console.error(err),
);
```

------------------------------------------------------------------------

## STEP 5 : Flow

```text
Next.js layout        embed.tawk.to           Tawk dashboard
     |                     |                        |
 inject loader ----------> widget bundle            |
 setAttributes(hash) ------------------------------> visitor identified
 visitor message ---------------------------------> agent reply
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use strategy="lazyOnload" so chat never blocks first paint
✓ Turn on Secure Mode and compute the visitor hash server-side
✓ Keep the API key server-only; the Property/Widget IDs stay public
✓ Restrict widget domains in admin to stop unauthorized embeds
✓ Mount the loader once, in the root layout, to avoid duplicates
✓ Position and theme the widget so it never covers key CTAs
✓ Pick Tawk when a zero-cost, unlimited-seat inbox is the goal
```
