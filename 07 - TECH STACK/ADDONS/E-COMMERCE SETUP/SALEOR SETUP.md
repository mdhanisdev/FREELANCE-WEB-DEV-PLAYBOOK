# SALEOR SETUP [ E-COMMERCE ]
------------------------------------------------------------------------

Saleor is a GraphQL-first, API-only commerce platform. Everything -
products, checkout, payments, promotions - is exposed through a single
GraphQL endpoint. You build the storefront in Next.js and talk to Saleor
Cloud (or a self-hosted instance) via typed queries.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add urql graphql
pnpm add -D @graphql-codegen/cli @graphql-codegen/client-preset
```

`urql` is a lightweight GraphQL client; codegen generates fully typed
documents from the Saleor schema.

------------------------------------------------------------------------

## STEP 2 : Configure Environment

Sign up for Saleor Cloud, create a project, and copy the GraphQL API URL.

```bash
# .env.local
NEXT_PUBLIC_SALEOR_API_URL=https://your-store.saleor.cloud/graphql/
SALEOR_APP_TOKEN=xxxxxxxxxxxxxxxx
```

> Windows note: the codegen CLI runs fine under Git Bash or PowerShell;
> no native build tools are required.

------------------------------------------------------------------------

## STEP 3 : Set Up Codegen

```ts
// codegen.ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: process.env.NEXT_PUBLIC_SALEOR_API_URL,
  documents: ["app/**/*.tsx", "lib/**/*.ts"],
  generates: {
    "./gql/": { preset: "client" },
  },
};
export default config;
```

```bash
pnpm graphql-codegen --config codegen.ts
```

------------------------------------------------------------------------

## STEP 4 : Create the Client and Query

```ts
// lib/saleor.ts
import { createClient, cacheExchange, fetchExchange } from "urql";

export const client = createClient({
  url: process.env.NEXT_PUBLIC_SALEOR_API_URL!,
  exchanges: [cacheExchange, fetchExchange],
});
```

```tsx
// app/catalog/page.tsx
import { client } from "@/lib/saleor";

const QUERY = /* GraphQL */ `
  query Products($channel: String!) {
    products(first: 12, channel: $channel) {
      edges { node { id name
        pricing { priceRange { start { gross { amount currency } } } } } }
    }
  }
`;

export default async function Catalog() {
  const { data } = await client
    .query(QUERY, { channel: "default-channel" })
    .toPromise();
  return (
    <ul className="grid grid-cols-3 gap-6 p-8">
      {data.products.edges.map(({ node }: any) => (
        <li key={node.id} className="rounded-lg border p-4">{node.name}</li>
      ))}
    </ul>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Channels and Checkout Model

```text
  +----------------------------------------------------+
  |                 Saleor GraphQL API                 |
  +----------------------------------------------------+
     |            |               |             |
  Channels    Products        Checkout       Payments
  (US, EU)    + Variants      (mutations)    (gateways)
     |
  Prices, currency, and availability are all channel-scoped:
  the same product can differ per channel.
```

Checkout is a sequence of GraphQL mutations: `checkoutCreate`,
`checkoutLinesAdd`, `checkoutPaymentCreate`, then `checkoutComplete`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always pass a channel; pricing is meaningless without one
✓ Generate types with codegen and commit the gql/ output
✓ Keep app tokens server-side; use permission-scoped tokens
✓ Cache read queries; use requestPolicy for freshness control
✓ Drive checkout entirely through GraphQL mutations
✓ Pin the Saleor API version your schema was generated against
✓ Use Saleor Apps/webhooks for async fulfilment, not polling
```
