# PINO SETUP [ STRUCTURED APPLICATION LOGGING ]
------------------------------------------------------------------------
## STEP 1 : Install

Pino is a low-overhead JSON logger. In dev you want pretty output; in
prod you want raw JSON that a log platform can index.

```bash
pnpm add pino
pnpm add -D pino-pretty
```

------------------------------------------------------------------------
## STEP 2 : Create the Base Logger

Centralise one logger. Set level by env, and redact secrets at the
source so they never reach disk or a log platform.

```ts
// app/lib/logger.ts
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  // Redact PII + secrets. Paths are matched in the log object.
  redact: {
    paths: [
      "req.headers.authorization",
      "req.headers.cookie",
      "*.password",
      "*.token",
      "user.email",
      "user.phone",
    ],
    censor: "[REDACTED]",
  },
  transport:
    process.env.NODE_ENV === "development"
      ? { target: "pino-pretty", options: { colorize: true } }
      : undefined, // prod: raw JSON to stdout
});
```

------------------------------------------------------------------------
## STEP 3 : Understand the Levels

```text
  fatal (60)  process is about to die
  error (50)  a request failed, needs attention
  warn  (40)  unexpected but recovered
  info  (30)  normal lifecycle events (default floor in prod)
  debug (20)  detailed flow, off in prod
  trace (10)  very noisy, local only
```

Only logs at or above `LOG_LEVEL` are emitted. Keep prod at `info`.

------------------------------------------------------------------------
## STEP 4 : Child Loggers Per Request

A child logger carries fixed fields (like a request id) so every line
from that request is correlated. Never bind PII into the child.

```ts
// app/lib/request-logger.ts
import { randomUUID } from "node:crypto";
import { logger } from "./logger";

export function requestLogger(route: string) {
  return logger.child({
    requestId: randomUUID(),
    route,
  });
}
```

```ts
// app/api/orders/route.ts
import { requestLogger } from "@/app/lib/request-logger";

export async function POST(req: Request) {
  const log = requestLogger("/api/orders");
  log.info({ step: "start" }, "creating order");

  try {
    const order = await createOrder(await req.json());
    log.info({ orderId: order.id }, "order created");
    return Response.json(order);
  } catch (err) {
    log.error({ err }, "order failed");
    return new Response("error", { status: 500 });
  }
}
```

------------------------------------------------------------------------
## STEP 5 : Where the Logs Go

Pino writes JSON to `stdout`. Do not manage log files yourself — let
the platform capture stdout and route it.

```text
  app --> logger.info() --> stdout (JSON line)
                              |
        +---------------------+----------------------+
        |                     |                      |
     Vercel logs        Docker/PM2            Axiom / Datadog
     (dashboard)        stdout capture        (ship + query)
```

------------------------------------------------------------------------
## STEP 6 : Log Messages, Not Objects You Regret

Pass structured fields as the first arg, a short message as the second.

```ts
// Good: queryable fields + human message, no PII
log.info({ userId: id, plan: "pro" }, "subscription upgraded");

// Bad: dumps the whole user, may include email/address
log.info(user, "subscription upgraded");
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ One base logger, imported everywhere
✓ Redact tokens, cookies, passwords, email at the source
✓ Log to stdout as JSON — never write log files by hand
✓ Use a child logger with a requestId per request
✓ Keep production at info; enable debug only when investigating
✓ Put queryable data in fields, a short string in the message
✓ Never log full request bodies or PII you do not need
✓ Attach errors as { err } so pino serialises the stack
```
