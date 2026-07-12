# SHOPIFY-HYDROGEN SETUP [ E-COMMERCE ]
------------------------------------------------------------------------

Hydrogen is Shopify's React framework for headless storefronts, powered by
the Storefront API. This guide wires a Hydrogen-style storefront into a
Next.js App Router project so you keep full control of the frontend while
Shopify handles catalog, checkout, and payments.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add @shopify/hydrogen-react graphql
pnpm add -D @types/react
```

`@shopify/hydrogen-react` gives you framework-agnostic components,
hooks, and a typed Storefront API client that runs in React Server
Components.

------------------------------------------------------------------------

## STEP 2 : Configure Environment

Create a custom app in the Shopify admin (Settings -> Apps and sales
channels -> Develop apps) and grant Storefront API scopes.

```bash
# .env.local
SHOPIFY_STORE_DOMAIN=your-store.myshopify.com
SHOPIFY_STOREFRONT_API_TOKEN=shpat_xxxxxxxxxxxxxxxx
SHOPIFY_API_VERSION=2025-07
```

> Windows note: use the built-in `.env.local` file; do not `export`
> variables in PowerShell. Next.js loads `.env.local` automatically.

------------------------------------------------------------------------

## STEP 3 : Create the Storefront Client

```ts
// lib/shopify.ts
import { createStorefrontClient } from "@shopify/hydrogen-react";

export const client = createStorefrontClient({
  storeDomain: process.env.SHOPIFY_STORE_DOMAIN!,
  privateStorefrontToken: process.env.SHOPIFY_STOREFRONT_API_TOKEN!,
  storefrontApiVersion: process.env.SHOPIFY_API_VERSION!,
});

export async function storefront<T>(query: string, variables = {}) {
  const res = await fetch(client.getStorefrontApiUrl(), {
    method: "POST",
    headers: client.getPrivateTokenHeaders(),
    body: JSON.stringify({ query, variables }),
    next: { revalidate: 60 },
  });
  const { data, errors } = await res.json();
  if (errors) throw new Error(JSON.stringify(errors));
  return data as T;
}
```

------------------------------------------------------------------------

## STEP 4 : Fetch Products in a Server Component

```tsx
// app/products/page.tsx
import { storefront } from "@/lib/shopify";

const QUERY = /* GraphQL */ `
  query Products {
    products(first: 12) {
      nodes { id title handle
        priceRange { minVariantPrice { amount currencyCode } } }
    }
  }
`;

export default async function ProductsPage() {
  const data = await storefront<{ products: { nodes: any[] } }>(QUERY);
  return (
    <ul className="grid grid-cols-3 gap-6 p-8">
      {data.products.nodes.map((p) => (
        <li key={p.id} className="rounded-lg border p-4">
          <h2 className="font-medium">{p.title}</h2>
          <p className="text-sm text-neutral-500">
            {p.priceRange.minVariantPrice.amount}{" "}
            {p.priceRange.minVariantPrice.currencyCode}
          </p>
        </li>
      ))}
    </ul>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Cart + Checkout Handoff

```text
   Next.js App Router (your UI)
            |
            v
   +--------------------+       cartCreate / cartLinesAdd
   |  Storefront API    | <-------------------------------+
   |  (GraphQL)         |                                 |
   +--------------------+                                 |
            |  checkoutUrl                                |
            v                                             |
   Shopify Hosted Checkout  ---> Payment ---> Order  -----+
```

Wrap client components in `<CartProvider>` from `@shopify/hydrogen-react`,
then redirect the customer to the returned `checkoutUrl`. Shopify owns PCI
compliance and payment capture, so you never touch card data.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep the Storefront token server-side; never expose private tokens
✓ Pin the API version (e.g. 2025-07) and upgrade quarterly
✓ Fetch catalog data in Server Components with next.revalidate
✓ Use CartProvider for optimistic client-side cart state
✓ Redirect to Shopify checkoutUrl instead of rebuilding checkout
✓ Cache product queries; invalidate via webhooks on product update
✓ Type Storefront responses with generated GraphQL types
```
