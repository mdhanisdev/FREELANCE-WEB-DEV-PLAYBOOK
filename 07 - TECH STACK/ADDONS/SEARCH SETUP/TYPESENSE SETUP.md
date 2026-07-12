# TYPESENSE SETUP [ SEARCH ]
------------------------------------------------------------------------

Typesense is an open-source, typo-tolerant search engine with a strict
typed schema and fast faceting. Self-host or use Typesense Cloud. It
sits between Meilisearch's simplicity and Algolia's feature depth.

------------------------------------------------------------------------

## STEP 1 : Run Typesense & Install

```bash
pnpm add typesense
```

```bash
# local instance (Windows: Docker Desktop / WSL2)
docker run -d --name typesense -p 8108:8108 \
  -v typesense-data:/data \
  typesense/typesense:27.1 \
  --data-dir /data --api-key=placeholder_api_key
```

```text
# .env.local
TYPESENSE_HOST=localhost
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=placeholder_api_key
```

------------------------------------------------------------------------

## STEP 2 : Server Client

```ts
// lib/search/client.ts
import Typesense from "typesense";

export const typesense = new Typesense.Client({
  nodes: [
    {
      host: process.env.TYPESENSE_HOST!,
      port: Number(process.env.TYPESENSE_PORT),
      protocol: process.env.TYPESENSE_PROTOCOL!,
    },
  ],
  apiKey: process.env.TYPESENSE_API_KEY!,
  connectionTimeoutSeconds: 5,
});
```

------------------------------------------------------------------------

## STEP 3 : Create A Collection (Typed Schema)

```ts
// lib/search/schema.ts
import { typesense } from "./client";

export async function createCollection() {
  await typesense.collections().create({
    name: "products",
    fields: [
      { name: "name", type: "string" },
      { name: "description", type: "string" },
      { name: "category", type: "string", facet: true },
      { name: "price", type: "float", facet: true },
    ],
    default_sorting_field: "price",
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Import Documents

```ts
// lib/search/import.ts
import { typesense } from "./client";

export async function importProducts(rows: Product[]) {
  await typesense
    .collections("products")
    .documents()
    .import(rows, { action: "upsert" });
}
```

------------------------------------------------------------------------

## STEP 5 : Search From A Route Handler

```ts
// app/api/search/route.ts
import { typesense } from "@/lib/search/client";

export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get("q") ?? "*";
  const res = await typesense
    .collections("products")
    .documents()
    .search({
      q,
      query_by: "name,description",
      filter_by: "price:<100",
      sort_by: "price:asc",
      per_page: 20,
    });
  return Response.json(res.hits);
}
```

------------------------------------------------------------------------

## STEP 6 : Scoped Search-Only Key

```ts
// generate a browser-safe key derived from a search-only parent key
const scoped = typesense.keys().generateScopedSearchKey(
  process.env.TYPESENSE_SEARCH_KEY!,
  { filter_by: "category:public" },
);
```

------------------------------------------------------------------------

## STEP 7 : Flow

```text
  DB rows          server               Typesense           client
  -------          ------               ---------           ------
  products --> documents.import --> [ products collection ]
                                          ^   |
   GET /api/search?q=... --> search() ----+   |
                                              v
                                    typo-tolerant + faceted hits
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Model the schema strictly — declare types and facet fields upfront
✓ Use action:"upsert" imports for idempotent re-indexing
✓ Always pass query_by; it defines which fields are searched
✓ Hand the browser a scoped search-only key, never the admin key
✓ Set default_sorting_field so ranked results have a stable tiebreak
✓ Pin the Typesense image version and mount a data volume
✓ Batch imports in chunks for large datasets
✓ On Windows run via Docker Desktop or WSL2, not a native binary
```
