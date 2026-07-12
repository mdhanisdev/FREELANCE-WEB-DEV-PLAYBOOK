# PGVECTOR SETUP [ AI ]
------------------------------------------------------------------------

pgvector adds a `vector` column type to Postgres so you can store
embeddings and run similarity search next to your relational data. This
is the RAG backbone: embed content, store it, retrieve by cosine match.

------------------------------------------------------------------------

## STEP 1 : Enable The Extension

```bash
pnpm add ai @ai-sdk/openai postgres
```

```sql
-- run once against your database
CREATE EXTENSION IF NOT EXISTS vector;
```

```text
# .env.local
DATABASE_URL=postgres://user:pass@localhost:5432/app
OPENAI_API_KEY=sk-placeholder
```

------------------------------------------------------------------------

## STEP 2 : Create The Embeddings Table

text-embedding-3-small produces 1536-dim vectors. Match the column size.

```sql
CREATE TABLE documents (
  id          bigserial PRIMARY KEY,
  content     text NOT NULL,
  embedding   vector(1536) NOT NULL
);

-- approximate-nearest-neighbor index for cosine distance
CREATE INDEX ON documents
  USING hnsw (embedding vector_cosine_ops);
```

------------------------------------------------------------------------

## STEP 3 : DB Client

```ts
// lib/db.ts
import postgres from "postgres";

export const sql = postgres(process.env.DATABASE_URL!);
```

------------------------------------------------------------------------

## STEP 4 : Embed & Insert (Ingest)

```ts
// lib/rag/ingest.ts
import { openai } from "@ai-sdk/openai";
import { embedMany } from "ai";
import { sql } from "@/lib/db";

export async function ingest(chunks: string[]) {
  const { embeddings } = await embedMany({
    model: openai.embedding("text-embedding-3-small"),
    values: chunks,
  });

  for (let i = 0; i < chunks.length; i++) {
    const vec = `[${embeddings[i].join(",")}]`; // pgvector literal
    await sql`
      INSERT INTO documents (content, embedding)
      VALUES (${chunks[i]}, ${vec})
    `;
  }
}
```

------------------------------------------------------------------------

## STEP 5 : Similarity Search (Retrieve)

The `<=>` operator is cosine distance — smaller means more similar.

```ts
// lib/rag/retrieve.ts
import { openai } from "@ai-sdk/openai";
import { embed } from "ai";
import { sql } from "@/lib/db";

export async function retrieve(query: string, k = 5) {
  const { embedding } = await embed({
    model: openai.embedding("text-embedding-3-small"),
    value: query,
  });
  const vec = `[${embedding.join(",")}]`;

  return sql<{ content: string; distance: number }[]>`
    SELECT content, embedding <=> ${vec} AS distance
    FROM documents
    ORDER BY distance ASC
    LIMIT ${k}
  `;
}
```

------------------------------------------------------------------------

## STEP 6 : Use Context In A Chat Route

```ts
// app/api/ask/route.ts
import { openai } from "@ai-sdk/openai";
import { streamText } from "ai";
import { retrieve } from "@/lib/rag/retrieve";

export async function POST(req: Request) {
  const { question } = await req.json();
  const hits = await retrieve(question);
  const context = hits.map((h) => h.content).join("\n---\n");

  const result = streamText({
    model: openai("gpt-4o"),
    system: `Answer using ONLY this context:\n${context}`,
    prompt: question,
  });
  return result.toUIMessageStreamResponse();
}
```

------------------------------------------------------------------------

## STEP 7 : RAG Pipeline

```text
  ingest                          query
  ------                          -----
  chunks -> embedMany -> INSERT   question -> embed
                    |                              |
                    v                              v
             documents(embedding) <== ORDER BY <=> ==> top-k
                                                        |
                                                        v
                                          streamText(system=context)
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep vector(N) dimensions exactly matching your embedding model
✓ Use an HNSW index with vector_cosine_ops for fast ANN search
✓ Chunk content to ~200-500 tokens with slight overlap before embedding
✓ Batch with embedMany to cut round trips and cost
✓ Store source metadata (url, chunk_id) to cite retrieved answers
✓ Normalize the distance metric across ingest and query (cosine)
✓ Re-embed everything if you change the embedding model
✓ On Windows run Postgres via Docker/WSL2 to get pgvector easily
```
