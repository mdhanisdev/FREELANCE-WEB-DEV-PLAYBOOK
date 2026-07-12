# AXIOM SETUP [ LOG SHIPPING + QUERYING ]
------------------------------------------------------------------------
## STEP 1 : When To Choose Axiom

Axiom is a cheap, high-volume log + event store with a fast query UI.
Choose it when you outgrow scrolling raw platform logs and want to
search, chart, and alert on structured events.

```text
  stdout logs        -> fine for a quick tail, hard to search at scale
  Axiom              -> ingest everything, query with APL, build alerts
  Use Axiom when you ask "how often did X happen last week?"
```

------------------------------------------------------------------------
## STEP 2 : Install next-axiom

For Next.js, `next-axiom` ships Vercel/console logs with zero transport
wiring. It is the simplest path on the App Router.

```bash
pnpm add next-axiom
```

```bash
# .env.local  (placeholders — never commit real tokens)
NEXT_PUBLIC_AXIOM_DATASET=my-app-logs
NEXT_PUBLIC_AXIOM_TOKEN=xaat-placeholder-token
```

------------------------------------------------------------------------
## STEP 3 : Wrap the Config

```ts
// next.config.ts
import { withAxiom } from "next-axiom";

export default withAxiom({
  // your existing Next.js config
});
```

------------------------------------------------------------------------
## STEP 4 : Log From Routes and Components

```ts
// app/api/checkout/route.ts
import { Logger } from "next-axiom";

export async function POST(req: Request) {
  const log = new Logger();
  log.info("checkout started", { plan: "pro" }); // no PII

  try {
    // ...work...
    log.info("checkout ok");
  } catch (err) {
    log.error("checkout failed", { err });
  } finally {
    await log.flush(); // ensure delivery before the function ends
  }
  return new Response("ok");
}
```

------------------------------------------------------------------------
## STEP 5 : Alternative — Pino Transport

If you already standardised on Pino, ship its JSON stream to Axiom with
`@axiomhq/pino` instead of adopting a second logger.

```bash
pnpm add @axiomhq/pino
```

```ts
// app/lib/logger.ts
import pino from "pino";

export const logger = pino(
  { level: "info" },
  pino.transport({
    target: "@axiomhq/pino",
    options: {
      dataset: process.env.AXIOM_DATASET,
      token: process.env.AXIOM_TOKEN,
    },
  })
);
```

------------------------------------------------------------------------
## STEP 6 : Query With APL

Axiom Processing Language reads like a pipeline. Query in the dashboard
or via the API.

```text
['my-app-logs']
| where level == "error"
| where _time > ago(24h)
| summarize count() by route
| sort by count_ desc
```

```text
  app --> next-axiom / pino transport --> Axiom dataset
                                             |
                                     APL query + charts
                                             |
                                     monitors --> alerts
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep the ingest token in an env var, never in git
✓ Use next-axiom for the simplest Next.js path
✓ If already on Pino, use the transport instead of two loggers
✓ Always await log.flush() in serverless routes
✓ Log structured fields so APL can group and chart them
✓ Never ship PII — redact before it reaches Axiom
✓ Build monitors on error counts, not on single lines
✓ Set dataset retention to match your compliance needs
```
