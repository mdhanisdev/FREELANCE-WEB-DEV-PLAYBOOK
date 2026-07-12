# VERCEL AI SDK SETUP [ AI ]
------------------------------------------------------------------------

The Vercel AI SDK is our default AI layer. One typed API over many
providers: streaming text, React hooks, structured objects validated by
Zod, and tool calling. It maps cleanly onto Next.js Route Handlers.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add ai @ai-sdk/openai @ai-sdk/react zod
```

```text
# .env.local
OPENAI_API_KEY=sk-placeholder
```

------------------------------------------------------------------------

## STEP 2 : Stream Text From A Route Handler

```ts
// app/api/chat/route.ts
import { openai } from "@ai-sdk/openai";
import { streamText, convertToModelMessages, type UIMessage } from "ai";

export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai("gpt-4o"),
    system: "You are a concise, helpful assistant.",
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

------------------------------------------------------------------------

## STEP 3 : Wire Up useChat On The Client

```tsx
// app/chat/page.tsx
"use client";
import { useChat } from "@ai-sdk/react";
import { useState } from "react";

export default function Chat() {
  const { messages, sendMessage, status } = useChat();
  const [input, setInput] = useState("");

  return (
    <div>
      {messages.map((m) => (
        <p key={m.id}>
          <b>{m.role}:</b>{" "}
          {m.parts.map((p, i) =>
            p.type === "text" ? <span key={i}>{p.text}</span> : null,
          )}
        </p>
      ))}
      <form
        onSubmit={(e) => {
          e.preventDefault();
          sendMessage({ text: input });
          setInput("");
        }}
      >
        <input value={input} onChange={(e) => setInput(e.target.value)} />
        <button disabled={status !== "ready"}>Send</button>
      </form>
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Structured Output With generateObject + Zod

```ts
// lib/ai/extract.ts
import { openai } from "@ai-sdk/openai";
import { generateObject } from "ai";
import { z } from "zod";

const Recipe = z.object({
  name: z.string(),
  ingredients: z.array(z.object({ item: z.string(), qty: z.string() })),
  steps: z.array(z.string()),
});

export async function extractRecipe(text: string) {
  const { object } = await generateObject({
    model: openai("gpt-4o"),
    schema: Recipe,
    prompt: `Extract a structured recipe from:\n${text}`,
  });
  return object; // fully typed + runtime-validated
}
```

------------------------------------------------------------------------

## STEP 5 : Tool Calling

```ts
// app/api/agent/route.ts
import { openai } from "@ai-sdk/openai";
import { streamText, tool, stepCountIs } from "ai";
import { z } from "zod";

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai("gpt-4o"),
    messages,
    stopWhen: stepCountIs(5), // allow multi-step tool loops
    tools: {
      getWeather: tool({
        description: "Get current weather for a city",
        inputSchema: z.object({ city: z.string() }),
        execute: async ({ city }) => {
          const r = await fetch(`https://api.example.com/wx?c=${city}`);
          return r.json();
        },
      }),
    },
  });

  return result.toUIMessageStreamResponse();
}
```

------------------------------------------------------------------------

## STEP 6 : Request Flow

```text
  client (useChat)        Route Handler            provider
  ---------------        -------------            --------
  sendMessage() ------>  streamText()  --------->  model
        ^                     |  tool call?
        |                     v
        |                execute tool -> feed result back
        |  SSE token stream   |  (loop up to stopWhen)
        +---------------------+  toUIMessageStreamResponse()
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep API keys server-side; only Route Handlers touch the provider
✓ Set maxDuration on streaming routes so they aren't killed early
✓ Validate every structured output with a Zod schema, not casts
✓ Use stopWhen/stepCountIs to bound multi-step tool loops
✓ Return toUIMessageStreamResponse() so useChat parses parts cleanly
✓ Swap providers by changing the model() call, not your app code
✓ Handle the status field to disable input while streaming
✓ Log token usage from the result for cost visibility
```
