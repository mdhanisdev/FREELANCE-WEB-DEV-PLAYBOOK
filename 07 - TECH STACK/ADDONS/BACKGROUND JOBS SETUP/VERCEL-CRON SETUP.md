# VERCEL CRON SETUP [ BACKGROUND JOBS ]
------------------------------------------------------------------------

Vercel Cron invokes a Route Handler on a schedule. Zero infrastructure:
you declare schedules in vercel.json and Vercel hits the endpoint. Best
for simple periodic work; not a queue and not for long-running jobs.

------------------------------------------------------------------------

## STEP 1 : Declare Schedules

```json
// vercel.json
{
  "crons": [
    { "path": "/api/cron/cleanup", "schedule": "0 3 * * *" },
    { "path": "/api/cron/digest", "schedule": "0 8 * * 1" }
  ]
}
```

Schedules use UTC. `0 3 * * *` = 03:00 UTC daily; `0 8 * * 1` = Mondays.

------------------------------------------------------------------------

## STEP 2 : Protect The Endpoint

Vercel sends `Authorization: Bearer <CRON_SECRET>`. Verify it so the
route cannot be triggered by the public.

```text
# .env (set in Vercel project settings)
CRON_SECRET=placeholder_cron_secret
```

------------------------------------------------------------------------

## STEP 3 : Write The Route Handler

```ts
// app/api/cron/cleanup/route.ts
import { NextRequest } from "next/server";

export async function GET(req: NextRequest) {
  const auth = req.headers.get("authorization");
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response("Unauthorized", { status: 401 });
  }

  const deleted = await db.session.deleteExpired();
  return Response.json({ ok: true, deleted });
}
```

------------------------------------------------------------------------

## STEP 4 : Control Runtime & Timeout

```ts
// same file — force Node runtime and raise the limit
export const runtime = "nodejs";
export const maxDuration = 60; // seconds (Pro plan; Hobby is lower)
export const dynamic = "force-dynamic";
```

------------------------------------------------------------------------

## STEP 5 : Verify Locally

Cron only fires in production, so invoke the route manually in dev:

```bash
pnpm dev

# simulate the Vercel cron request (Git Bash / PowerShell)
curl -H "Authorization: Bearer placeholder_cron_secret" \
  http://localhost:3000/api/cron/cleanup
```

------------------------------------------------------------------------

## STEP 6 : Deploy & Confirm

```bash
pnpm dlx vercel --prod
```

After deploy, open Vercel Dashboard -> Project -> Cron Jobs to see the
next run time and the log of each invocation.

------------------------------------------------------------------------

## STEP 7 : How It Fires

```text
  Vercel Scheduler          your deployment          route handler
  ----------------          ---------------          -------------
  cron matches (UTC)
        |  GET /api/cron/cleanup
        |  Authorization: Bearer CRON_SECRET
        v
   edge/network ---------->  Route Handler ------->  verify secret
                                                     do work (< maxDuration)
                                                     return 200 JSON
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always verify the Bearer CRON_SECRET before doing any work
✓ Keep each job under maxDuration — offload heavy work to a queue
✓ Remember schedules are UTC, not local time
✓ Return a JSON summary so logs show what each run did
✓ Make handlers idempotent; a run may retry or overlap on redeploy
✓ Use GET handlers — Vercel Cron issues GET requests
✓ Hobby plan limits cron count/frequency; check plan limits first
✓ For fan-out or retries, have the cron enqueue into Inngest/BullMQ
```
