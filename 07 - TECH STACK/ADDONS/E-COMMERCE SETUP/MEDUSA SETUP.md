# MEDUSA SETUP [ E-COMMERCE ]
------------------------------------------------------------------------

Medusa is an open-source, self-hosted commerce engine (Node.js + Postgres).
You run the backend yourself and drive it from a Next.js App Router
storefront through the Store REST/JS API. This gives you full data
ownership with no per-transaction platform fee.

------------------------------------------------------------------------

## STEP 1 : Scaffold the Backend

```bash
pnpm dlx create-medusa-app@latest my-store
cd my-store
pnpm dev
```

The CLI provisions the server, admin dashboard, and a Postgres schema. The
backend defaults to `http://localhost:9000`, admin to `/app`.

> Windows note: Medusa needs a local Postgres. Easiest path is Docker
> Desktop (`docker run -e POSTGRES_PASSWORD=pg -p 5432:5432 postgres`),
> since native Postgres services can conflict with WSL ports.

------------------------------------------------------------------------

## STEP 2 : Install the JS SDK in the Storefront

```bash
pnpm add @medusajs/js-sdk @medusajs/types
```

```bash
# .env.local
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_xxxxxxxxxxxxxxxx
```

Create a publishable API key in the admin under Settings -> Publishable
API Keys and scope it to a sales channel.

------------------------------------------------------------------------

## STEP 3 : Create the Client

```ts
// lib/medusa.ts
import Medusa from "@medusajs/js-sdk";

export const medusa = new Medusa({
  baseUrl: process.env.NEXT_PUBLIC_MEDUSA_BACKEND_URL!,
  publishableKey: process.env.NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY!,
});
```

------------------------------------------------------------------------

## STEP 4 : List Products

```tsx
// app/store/page.tsx
import { medusa } from "@/lib/medusa";

export default async function StorePage() {
  const { products } = await medusa.store.product.list({ limit: 12 });
  return (
    <div className="grid grid-cols-3 gap-6 p-8">
      {products.map((p) => (
        <article key={p.id} className="rounded-lg border p-4">
          <h2 className="font-medium">{p.title}</h2>
          <p className="text-sm text-neutral-500">{p.description}</p>
        </article>
      ))}
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Architecture Overview

```text
  +-------------------+        +-----------------------+
  |  Next.js (UI)     | -----> |  Medusa Backend       |
  |  @medusajs/js-sdk |  REST  |  (Node.js server)     |
  +-------------------+        +----------+------------+
                                          |
                          +---------------+---------------+
                          |               |               |
                       Postgres        Redis         Admin (/app)
                     (products,       (events,       React
                      carts, orders)   workflows)    dashboard
```

Carts, regions, and orders all live in Postgres. Add a Redis module for
event/workflow durability in production.

------------------------------------------------------------------------

## STEP 6 : Cart Flow

```ts
const { cart } = await medusa.store.cart.create({ region_id });
await medusa.store.cart.createLineItem(cart.id, {
  variant_id: variantId,
  quantity: 1,
});
await medusa.store.cart.complete(cart.id); // creates the order
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Run migrations (pnpm medusa db:migrate) on every deploy
✓ Use a managed Postgres + Redis in production, never local disk
✓ Scope publishable keys to a single sales channel
✓ Keep secret admin API keys server-side only
✓ Add a payment provider module (Stripe) before going live
✓ Back up the database and store event workflows in Redis
✓ Version-lock @medusajs packages together to avoid drift
```
