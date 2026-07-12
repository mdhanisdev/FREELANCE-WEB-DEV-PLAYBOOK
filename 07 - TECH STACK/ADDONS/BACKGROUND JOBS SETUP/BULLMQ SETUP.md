# BULLMQ SETUP [ BACKGROUND JOBS ]
------------------------------------------------------------------------

BullMQ is a Redis-backed queue. You own the infrastructure: a Redis
instance, a producer that enqueues jobs, and a worker process that
consumes them. Best when you already run Redis and want full control.

------------------------------------------------------------------------

## STEP 1 : Install & Provision Redis

```bash
pnpm add bullmq ioredis
```

```bash
# local Redis via Docker (Windows: use Docker Desktop / WSL2)
docker run -d --name redis -p 6379:6379 redis:7
```

```text
# .env.local
REDIS_URL=redis://localhost:6379
```

------------------------------------------------------------------------

## STEP 2 : Shared Connection

```ts
// lib/queue/connection.ts
import { Redis } from "ioredis";

// maxRetriesPerRequest MUST be null for BullMQ blocking commands
export const connection = new Redis(process.env.REDIS_URL!, {
  maxRetriesPerRequest: null,
});
```

------------------------------------------------------------------------

## STEP 3 : Define The Queue (Producer)

```ts
// lib/queue/email.queue.ts
import { Queue } from "bullmq";
import { connection } from "./connection";

export type EmailJob = { to: string; template: string };

export const emailQueue = new Queue<EmailJob>("email", {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: "exponential", delay: 2000 },
    removeOnComplete: 1000,
    removeOnFail: 5000,
  },
});
```

------------------------------------------------------------------------

## STEP 4 : Enqueue From A Route

```ts
// app/api/notify/route.ts
import { emailQueue } from "@/lib/queue/email.queue";

export async function POST(req: Request) {
  const body = await req.json();
  await emailQueue.add("welcome", {
    to: "user@example.com",
    template: "welcome",
  });
  return Response.json({ queued: true });
}
```

------------------------------------------------------------------------

## STEP 5 : The Worker Process

The worker runs OUTSIDE Next.js as its own long-lived Node process.

```ts
// worker/email.worker.ts
import { Worker } from "bullmq";
import { connection } from "@/lib/queue/connection";
import type { EmailJob } from "@/lib/queue/email.queue";

const worker = new Worker<EmailJob>(
  "email",
  async (job) => {
    await mailer.send({ to: job.data.to, template: job.data.template });
  },
  { connection, concurrency: 10 },
);

worker.on("completed", (job) => console.log(`done ${job.id}`));
worker.on("failed", (job, err) => console.error(`fail ${job?.id}`, err));
```

------------------------------------------------------------------------

## STEP 6 : Repeatable (Cron) Jobs

```ts
await emailQueue.add(
  "digest",
  { to: "user@example.com", template: "digest" },
  { repeat: { pattern: "0 8 * * *" } }, // 08:00 daily
);
```

------------------------------------------------------------------------

## STEP 7 : Run It

```bash
# terminal 1 — app
pnpm dev

# terminal 2 — worker (via tsx)
pnpm add -D tsx
pnpm tsx worker/email.worker.ts
```

------------------------------------------------------------------------

## STEP 8 : Architecture

```text
  Next.js route        Redis (BullMQ)         worker process
  -------------        --------------         --------------
  queue.add(job) --->  [ waiting list ]
                            |
                            v
                       [ active ] ---------->  process(job)
                            |  ok -> completed
                            |  throw -> retry (exp backoff)
                            v
                       [ failed ] after N attempts
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always set maxRetriesPerRequest: null on the BullMQ connection
✓ Run workers as a separate process/container, never inside a route
✓ Set removeOnComplete/removeOnFail so Redis memory stays bounded
✓ Make job handlers idempotent — retries re-run the whole handler
✓ Tune concurrency to match downstream capacity, not CPU count
✓ Use one Queue name per job type; keep payloads small + typed
✓ On Windows run Redis under WSL2 or Docker Desktop, not native
✓ Add a QueueEvents listener or Bull Board for observability
```
