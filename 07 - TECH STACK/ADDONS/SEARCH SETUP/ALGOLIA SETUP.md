# ALGOLIA SETUP [ SEARCH ]
------------------------------------------------------------------------

Algolia is a fully hosted search API with best-in-class relevance,
faceting, and a mature InstantSearch UI kit. Zero infra to run; you push
records and query from the browser with a search-only key.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add algoliasearch
pnpm add react-instantsearch instantsearch.js
```

```text
# .env.local
ALGOLIA_APP_ID=PLACEHOLDERAPPID
ALGOLIA_ADMIN_KEY=placeholder_admin_key           # server only
NEXT_PUBLIC_ALGOLIA_APP_ID=PLACEHOLDERAPPID
NEXT_PUBLIC_ALGOLIA_SEARCH_KEY=placeholder_search_key  # public, read-only
```

------------------------------------------------------------------------

## STEP 2 : Server Client (Admin)

```ts
// lib/search/admin.ts
import { algoliasearch } from "algoliasearch";

// admin key can write — keep it server-side ONLY
export const admin = algoliasearch(
  process.env.ALGOLIA_APP_ID!,
  process.env.ALGOLIA_ADMIN_KEY!,
);
```

------------------------------------------------------------------------

## STEP 3 : Push Records & Configure Settings

```ts
// lib/search/index.ts
import { admin } from "./admin";

export async function reindex(products: Product[]) {
  await admin.setSettings({
    indexName: "products",
    indexSettings: {
      searchableAttributes: ["name", "description", "tags"],
      attributesForFaceting: ["filterOnly(category)", "price"],
    },
  });
  // objectID is required and must be unique per record
  await admin.saveObjects({
    indexName: "products",
    objects: products.map((p) => ({ objectID: p.id, ...p })),
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Search From A Route Handler

```ts
// app/api/search/route.ts
import { admin } from "@/lib/search/admin";

export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get("q") ?? "";
  const { results } = await admin.search({
    requests: [{ indexName: "products", query: q, hitsPerPage: 20 }],
  });
  return Response.json(results[0].hits);
}
```

------------------------------------------------------------------------

## STEP 5 : InstantSearch UI (Client, Search Key)

```tsx
// app/search/page.tsx
"use client";
import { liteClient } from "algoliasearch/lite";
import { InstantSearch, SearchBox, Hits } from "react-instantsearch";

const client = liteClient(
  process.env.NEXT_PUBLIC_ALGOLIA_APP_ID!,
  process.env.NEXT_PUBLIC_ALGOLIA_SEARCH_KEY!,
);

export default function Search() {
  return (
    <InstantSearch searchClient={client} indexName="products">
      <SearchBox />
      <Hits hitComponent={({ hit }) => <div>{(hit as any).name}</div>} />
    </InstantSearch>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Key Roles

```text
  ADMIN KEY (server)              SEARCH KEY (public/browser)
  ------------------              ---------------------------
  saveObjects / setSettings       query only
  reindex, delete                 rate-limited, read-only
        |                                 |
        v                                 v
   build index  ------------------>  InstantSearch UI
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Never expose the admin key — only NEXT_PUBLIC_* search key ships
✓ Every record needs a stable unique objectID
✓ Use saveObjects batches; prefer partial updates for small changes
✓ Define attributesForFaceting to enable filters and facets
✓ Keep search analytics on to tune ranking with real queries
✓ Use replicas for alternate sort orders (price asc/desc)
✓ Rebuild via a tmp index + moveIndex for zero-downtime reindex
✓ No local server needed — same setup on Windows, macOS, Linux
```
