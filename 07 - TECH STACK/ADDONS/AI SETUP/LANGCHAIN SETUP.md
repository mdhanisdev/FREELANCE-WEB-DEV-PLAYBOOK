# LANGCHAIN SETUP [ AI ]
------------------------------------------------------------------------

LangChain.js composes LLM calls, prompts, retrievers, and tools into
runnable chains and agents. Reach for it when you need orchestration —
RAG pipelines, multi-tool agents — beyond a single model call.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add langchain @langchain/core @langchain/openai zod
```

```text
# .env.local
OPENAI_API_KEY=sk-placeholder
```

------------------------------------------------------------------------

## STEP 2 : A Basic LCEL Chain

LCEL (LangChain Expression Language) pipes runnables with `.pipe()`.

```ts
// lib/lc/summarize.ts
import { ChatOpenAI } from "@langchain/openai";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";

const model = new ChatOpenAI({ model: "gpt-4o", temperature: 0 });
const prompt = ChatPromptTemplate.fromMessages([
  ["system", "Summarize in one sentence."],
  ["human", "{text}"],
]);

export const summarizeChain = prompt
  .pipe(model)
  .pipe(new StringOutputParser());
```

------------------------------------------------------------------------

## STEP 3 : Stream From A Route Handler

```ts
// app/api/summarize/route.ts
import { summarizeChain } from "@/lib/lc/summarize";
import { LangChainAdapter } from "ai";

export async function POST(req: Request) {
  const { text } = await req.json();
  const stream = await summarizeChain.stream({ text });
  return LangChainAdapter.toDataStreamResponse(stream);
}
```

------------------------------------------------------------------------

## STEP 4 : Structured Output With Zod

```ts
// lib/lc/classify.ts
import { ChatOpenAI } from "@langchain/openai";
import { z } from "zod";

const schema = z.object({
  sentiment: z.enum(["positive", "neutral", "negative"]),
  topics: z.array(z.string()),
});

const model = new ChatOpenAI({ model: "gpt-4o" });
const structured = model.withStructuredOutput(schema);

export function classify(text: string) {
  return structured.invoke(text); // returns typed object
}
```

------------------------------------------------------------------------

## STEP 5 : A Tool-Calling Agent

```ts
// lib/lc/agent.ts
import { ChatOpenAI } from "@langchain/openai";
import { tool } from "@langchain/core/tools";
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { z } from "zod";

const search = tool(
  async ({ q }) => (await fetch(`https://api.example.com/s?q=${q}`)).text(),
  {
    name: "search",
    description: "Search the knowledge base",
    schema: z.object({ q: z.string() }),
  },
);

export const agent = createReactAgent({
  llm: new ChatOpenAI({ model: "gpt-4o" }),
  tools: [search],
});
```

```bash
pnpm add @langchain/langgraph
```

------------------------------------------------------------------------

## STEP 6 : Chain Composition

```text
  input {text}
      |
      v
  ChatPromptTemplate  --format-->  ChatOpenAI  --tokens-->  OutputParser
      |                                |                         |
      +------------ .pipe() -----------+---------- .pipe() -------+
                                                             string result
```

------------------------------------------------------------------------

## STEP 7 : Run

```bash
pnpm dev
# POST to http://localhost:3000/api/summarize with { "text": "..." }
```

On Windows use PowerShell/Git Bash; no platform-specific setup needed.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Prefer LCEL .pipe() composition over ad-hoc imperative glue
✓ Use withStructuredOutput + Zod instead of parsing raw strings
✓ Keep model temperature low (0) for extraction/classification
✓ Stream to the client via LangChainAdapter for AI SDK interop
✓ Use LangGraph prebuilt agents rather than the legacy AgentExecutor
✓ Keep provider keys server-side in Route Handlers
✓ Set request timeouts + maxRetries on the ChatOpenAI client
✓ Trace chains with LangSmith in staging to debug prompt flow
```
