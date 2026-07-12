# MASTRA SETUP [ AI ]
------------------------------------------------------------------------

Mastra is a TypeScript agent framework: typed agents, tools, workflows,
and memory in one place. It builds on the Vercel AI SDK, so providers
and streaming feel familiar while adding orchestration and persistence.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @mastra/core @ai-sdk/openai zod
```

```text
# .env.local
OPENAI_API_KEY=sk-placeholder
```

------------------------------------------------------------------------

## STEP 2 : Define A Tool

```ts
// src/mastra/tools/weather.ts
import { createTool } from "@mastra/core/tools";
import { z } from "zod";

export const weatherTool = createTool({
  id: "get-weather",
  description: "Get current weather for a city",
  inputSchema: z.object({ city: z.string() }),
  outputSchema: z.object({ tempC: z.number(), summary: z.string() }),
  execute: async ({ context }) => {
    const r = await fetch(`https://api.example.com/wx?c=${context.city}`);
    return r.json();
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Define An Agent

```ts
// src/mastra/agents/assistant.ts
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { weatherTool } from "../tools/weather";

export const assistant = new Agent({
  name: "assistant",
  instructions: "You are a concise assistant. Use tools when helpful.",
  model: openai("gpt-4o"),
  tools: { weatherTool },
});
```

------------------------------------------------------------------------

## STEP 4 : Register The Mastra Instance

```ts
// src/mastra/index.ts
import { Mastra } from "@mastra/core";
import { assistant } from "./agents/assistant";

export const mastra = new Mastra({
  agents: { assistant },
});
```

------------------------------------------------------------------------

## STEP 5 : Call The Agent From A Route

```ts
// app/api/assistant/route.ts
import { mastra } from "@/src/mastra";

export async function POST(req: Request) {
  const { prompt } = await req.json();
  const agent = mastra.getAgent("assistant");
  const stream = await agent.stream(prompt);
  return stream.toUIMessageStreamResponse();
}
```

------------------------------------------------------------------------

## STEP 6 : A Workflow (Sequenced Steps)

```ts
// src/mastra/workflows/onboard.ts
import { createWorkflow, createStep } from "@mastra/core/workflows";
import { z } from "zod";

const enrich = createStep({
  id: "enrich",
  inputSchema: z.object({ email: z.string() }),
  outputSchema: z.object({ score: z.number() }),
  execute: async ({ inputData }) => ({ score: score(inputData.email) }),
});

export const onboard = createWorkflow({
  id: "onboard",
  inputSchema: z.object({ email: z.string() }),
  outputSchema: z.object({ score: z.number() }),
})
  .then(enrich)
  .commit();
```

------------------------------------------------------------------------

## STEP 7 : Dev Playground

```bash
pnpm dlx mastra dev
# opens a local playground to chat with agents + inspect traces
```

On Windows run from PowerShell or Git Bash; the command is the same.

------------------------------------------------------------------------

## STEP 8 : Component Map

```text
   Route Handler
        |
        v
     Mastra ----> Agent (instructions + model)
                    |         |
                    |         +--> Tools (typed in/out via Zod)
                    +--> Memory (threads, persisted)
     Workflows ---> Steps (.then / .parallel / .branch)
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Give every tool strict Zod input AND output schemas
✓ Keep agent instructions short, specific, and behavior-focused
✓ Register all agents/workflows on one Mastra instance for discovery
✓ Use workflows for deterministic multi-step logic, agents for open loops
✓ Enable memory only where you need conversation continuity
✓ Keep provider keys server-side; the browser never sees them
✓ Use the dev playground to trace tool calls before shipping
✓ Return toUIMessageStreamResponse() to reuse AI SDK UI hooks
```
