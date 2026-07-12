# SNIPCART SETUP [ E-COMMERCE ]
------------------------------------------------------------------------

Snipcart bolts a full cart and checkout onto any existing site using HTML
data attributes. There is no product database to run: you annotate buttons
with price/name metadata and Snipcart validates against your site at
checkout. Ideal for content sites that need a light store.

------------------------------------------------------------------------

## STEP 1 : Get Your API Key

Create an account at snipcart.com, then copy the public API key from
Account -> API Keys. There is a Test key and a Live key.

```bash
# .env.local
NEXT_PUBLIC_SNIPCART_KEY=your_public_api_key
```

Snipcart is script-based, so no pnpm package is strictly required, but a
typed helper package keeps things tidy.

```bash
pnpm add @snipcart/react-snipcart
```

------------------------------------------------------------------------

## STEP 2 : Load the Snipcart Script

```tsx
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <div
          hidden
          id="snipcart"
          data-api-key={process.env.NEXT_PUBLIC_SNIPCART_KEY}
        />
        <Script
          src="https://cdn.snipcart.com/themes/v3.7.1/default/snipcart.js"
          strategy="afterInteractive"
        />
        <link
          rel="stylesheet"
          href="https://cdn.snipcart.com/themes/v3.7.1/default/snipcart.css"
        />
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Add a Buy Button

```tsx
// components/BuyButton.tsx
export function BuyButton() {
  return (
    <button
      className="snipcart-add-item rounded bg-black px-4 py-2 text-white"
      data-item-id="tshirt-001"
      data-item-name="Classic Tee"
      data-item-price="29.00"
      data-item-url="/products/classic-tee"
      data-item-description="100% cotton"
    >
      Add to cart
    </button>
  );
}
```

At checkout Snipcart crawls `data-item-url` to confirm the price matches,
preventing tampering. The URL must be publicly reachable.

------------------------------------------------------------------------

## STEP 4 : How Validation Works

```text
  Buy button (data-item-*)
        |
        v
  Snipcart cart overlay
        |
        |  fetches data-item-url and re-reads the
        |  data-item-price attribute on that page
        v
  Price match?  --no--> checkout blocked
        |
        yes
        v
  Hosted checkout + payment gateway (Stripe, etc.)
```

------------------------------------------------------------------------

## STEP 5 : Webhooks (Optional)

Point Snipcart -> Webhooks at a route handler to fulfil orders.

```ts
// app/api/snipcart/route.ts
export async function POST(req: Request) {
  const event = await req.json();
  if (event.eventName === "order.completed") {
    // fulfil, email receipt, update inventory
  }
  return Response.json({ ok: true });
}
```

> Windows note: nothing to install locally; test with the Test API key
> and Snipcart's built-in test card `4242 4242 4242 4242`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep data-item-price identical on button and crawled page
✓ Use isolated Test and Live keys per environment
✓ Ensure every data-item-url is publicly crawlable
✓ Verify webhook signatures before fulfilling orders
✓ Load snipcart.js with strategy="afterInteractive"
✓ Pin the Snipcart theme version in the CDN URL
✓ Never expose a secret key; the API key here is public by design
```
