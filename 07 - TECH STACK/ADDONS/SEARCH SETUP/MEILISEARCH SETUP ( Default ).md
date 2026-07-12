# MEILISEARCH SETUP [ SEARCH ]
------------------------------------------------------------------------

Meilisearch is our default search engine: instant, typo-tolerant search
with tiny config. Self-host or use Meilisearch Cloud. You index JSON
documents and query them with sub-50ms responses out of the box.

------------------------------------------------------------------------

## STEP 1 : Run Meilisearch & Install The Client

```bash
pnpm add meilisearch
```

```bash
# local instance via Docker (Windows: Docker Desktop / WSL2)
docker run -d --name meili -p 7700:7700 \
  -e MEILI_MASTER_KEY=placeholder_master_key \
  getmeili/meilisearch:v1.10
```

```text
# .env.local
MEILI_HOST=http://localhost:7700
MEILI_MASTER_KEY=placeholder_master_key
```

------------------------------------------------------------------------

## STEP 2 : Server-Side Client

Use the master key ONLY on the server; never ship it to the browser.

```ts
// lib/search/client.ts
import { MeiliSearch } from "meilisearch";

export const meili = new MeiliSearch({
  host: process.env.MEILI_HOST!,
  apiKey: process.env.MEILI_MASTER_KEY!,
});
```

------------------------------------------------------------------------

## STEP 3 : Configure An Index

```ts
// lib/search/setup.ts
import { meili } from "./client";

export async function configure() {
  const index = meili.index("products");
  await index.updateSettings({
    searchableAttributes: ["name", "description", "tags"],
    filterableAttributes: ["category", "price"],
    sortableAttributes: ["price", "createdAt"],
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Index Documents

```ts
// lib/search/index-products.ts
import { meili } from "./client";

export async function indexProducts(rows: Product[]) {
  const index = meili.index("products");
  // primaryKey must be a unique field named "id" here
  await index.addDocuments(rows, { primaryKey: "id" });
}
```

------------------------------------------------------------------------

## STEP 5 : Search From A Route Handler

```ts
// app/api/search/route.ts
import { meili } from "@/lib/search/client";

export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get("q") ?? "";
  const res = await meili.index("products").search(q, {
    limit: 20,
    filter: "price < 100",
    sort: ["price:asc"],
    attributesToHighlight: ["name"],
  });
  return Response.json(res.hits);
}
```

------------------------------------------------------------------------

## STEP 6 : Scoped Search Key For The Client

Generate a tenant-scoped, search-only key server-side so the browser
never holds the master key.

```ts
const searchKey = await meili.createKey({
  actions: ["search"],
  indexes: ["products"],
  expiresAt: null,
  description: "public search",
});
```

------------------------------------------------------------------------

## STEP 7 : Flow

```text
  DB rows        server                Meilisearch          client
  -------        ------                -----------          ------
  products --> addDocuments() -----> [ products index ]
                                          ^   |
   GET /api/search?q=... --> search() ----+   |
                                              v
                                     ranked, typo-tolerant hits
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep the master key server-only; hand clients scoped search keys
✓ Declare searchable/filterable/sortable attributes explicitly
✓ Batch addDocuments (1k-10k per call) for fast bulk indexing
✓ Re-index incrementally on writes, not a full rebuild each time
✓ Use filter + sort instead of post-filtering hits in your app
✓ Pin the Meilisearch image version for reproducible deploys
✓ On Windows run the engine via Docker Desktop or WSL2
✓ Monitor task queue via getTask for large indexing operations
```
