# TRIGGER.DEV SETUP [ BACKGROUND JOBS ]
------------------------------------------------------------------------

Trigger.dev runs long, durable tasks with no timeouts, built-in retries,
concurrency controls, and observability. Tasks are plain functions you
deploy; the Trigger.dev cloud (or self-hosted) orchestrates execution.

------------------------------------------------------------------------

## STEP 1 : Install & Init

```bash
pnpm add @trigger.dev/sdk@latest
pnpm dlx trigger.dev@latest init
```

```text
# .env.local
TRIGGER_SECRET_KEY=tr_dev_placeholder
TRIGGER_PROJECT_ID=proj_placeholder
```

------------------------------------------------------------------------

## STEP 2 : Configure The Project

```ts
// trigger.config.ts
import { defineConfig } from "@trigger.dev/sdk/v3";

export default defineConfig({
  project: process.env.TRIGGER_PROJECT_ID!,
  dirs: ["./src/trigger"],
  retries: {
    default: { maxAttempts: 3, factor: 2, minTimeoutInMs: 1000 },
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Define A Task

```ts
// src/trigger/process-video.ts
import { task, logger } from "@trigger.dev/sdk/v3";

export const processVideo = task({
  id: "process-video",
  maxDuration: 600, // seconds
  run: async (payload: { videoId: string }, { ctx }) => {
    logger.info("processing", { videoId: payload.videoId, run: ctx.run.id });
    const result = await transcode(payload.videoId);
    return { url: result.url };
  },
});
```

------------------------------------------------------------------------

## STEP 4 : Add A Scheduled (Cron) Task

```ts
// src/trigger/daily-report.ts
import { schedules } from "@trigger.dev/sdk/v3";

export const dailyReport = schedules.task({
  id: "daily-report",
  cron: "0 6 * * *", // 06:00 UTC daily
  run: async () => {
    await buildAndEmailReport("user@example.com");
  },
});
```

------------------------------------------------------------------------

## STEP 5 : Trigger Tasks From Your App

```ts
// app/api/upload/route.ts
import { tasks } from "@trigger.dev/sdk/v3";
import type { processVideo } from "@/trigger/process-video";

export async function POST(req: Request) {
  const { videoId } = await req.json();
  // type-safe handle — does NOT block on completion
  const handle = await tasks.trigger<typeof processVideo>(
    "process-video",
    { videoId },
  );
  return Response.json({ runId: handle.id });
}
```

------------------------------------------------------------------------

## STEP 6 : Develop & Deploy

```bash
# local dev — hot-reloads tasks against the cloud
pnpm dlx trigger.dev@latest dev

# ship to your environment
pnpm dlx trigger.dev@latest deploy
```

On Windows run these from PowerShell or Git Bash; behavior is identical.

------------------------------------------------------------------------

## STEP 7 : Execution Model

```text
  Next.js route          Trigger.dev              your task
  -------------          -----------              ---------
  tasks.trigger() --->  [ durable queue ]
                             |
                             v
                        run task fn ----------->  run(payload)
                             |  retries + backoff
                             |  no execution timeout
                             v
                        dashboard: logs, replays, metrics
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Import task types with `import type` for end-to-end payload safety
✓ Split big jobs into subtasks and use triggerAndWait for fan-out
✓ Set maxDuration so runaway runs are killed, not billed forever
✓ Use logger.* instead of console.log to get structured run logs
✓ Keep TRIGGER_SECRET_KEY server-side only
✓ Return small serializable results; store large output in blob/db
✓ Tune retries per task rather than relying on the global default
✓ Test locally with `dev` before every `deploy`
```
