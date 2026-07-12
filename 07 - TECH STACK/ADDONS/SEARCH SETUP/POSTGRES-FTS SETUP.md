# POSTGRES FTS SETUP [ SEARCH ]
------------------------------------------------------------------------

Postgres Full-Text Search needs no extra service: `tsvector`, `tsquery`,
and a GIN index give you ranked, stemmed search inside the database you
already run. Ideal when you want search without new infrastructure.

------------------------------------------------------------------------

## STEP 1 : Add A Generated tsvector Column

A generated column keeps the search vector in sync automatically.

```sql
ALTER TABLE products
  ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(name, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(description, '')), 'B')
  ) STORED;
```

------------------------------------------------------------------------

## STEP 2 : Index It With GIN

```sql
CREATE INDEX products_search_idx
  ON products USING gin (search_vector);
```

------------------------------------------------------------------------

## STEP 3 : DB Client

```bash
pnpm add postgres
```

```ts
// lib/db.ts
import postgres from "postgres";
export const sql = postgres(process.env.DATABASE_URL!);
```

```text
# .env.local
DATABASE_URL=postgres://user:pass@localhost:5432/app
```

------------------------------------------------------------------------

## STEP 4 : Ranked Query With websearch_to_tsquery

`websearch_to_tsquery` accepts Google-style input ("phrase" -exclude).

```ts
// lib/search/query.ts
import { sql } from "@/lib/db";

export async function searchProducts(q: string, limit = 20) {
  return sql<{ id: number; name: string; rank: number }[]>`
    SELECT id, name,
           ts_rank(search_vector, websearch_to_tsquery('english', ${q})) AS rank
    FROM products
    WHERE search_vector @@ websearch_to_tsquery('english', ${q})
    ORDER BY rank DESC
    LIMIT ${limit}
  `;
}
```

------------------------------------------------------------------------

## STEP 5 : Route Handler

```ts
// app/api/search/route.ts
import { searchProducts } from "@/lib/search/query";

export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get("q") ?? "";
  if (!q.trim()) return Response.json([]);
  return Response.json(await searchProducts(q));
}
```

------------------------------------------------------------------------

## STEP 6 : Highlight Matches

```ts
const rows = await sql`
  SELECT ts_headline('english', description,
           websearch_to_tsquery('english', ${q})) AS snippet
  FROM products
  WHERE search_vector @@ websearch_to_tsquery('english', ${q})
`;
```

------------------------------------------------------------------------

## STEP 7 : How It Matches

```text
  document text                  query text
  -------------                  ----------
  to_tsvector('english')         websearch_to_tsquery('english')
        |                                |
        v                                v
   'run':1 'fast':2  ...   @@   'run' & 'fast'   --> match?
        |                                |
        +---- ts_rank (weighted A/B) ----+ --> ORDER BY rank DESC
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use a STORED generated tsvector column so it can never drift
✓ Weight fields with setweight (A=title, B=body) for better ranking
✓ Always back the tsvector with a GIN index
✓ Prefer websearch_to_tsquery for user input — it never throws on syntax
✓ Pick one language config consistently for index and query
✓ Use ts_headline for snippets, but only on the returned rows
✓ For fuzzy/typo tolerance add pg_trgm; FTS alone stems, not corrects
✓ On Windows run Postgres via Docker/WSL2 for parity with prod
```
