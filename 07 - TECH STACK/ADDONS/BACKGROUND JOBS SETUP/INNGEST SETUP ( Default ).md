# INNGEST SETUP [ BACKGROUND JOBS ]
------------------------------------------------------------------------

Inngest is our default background-jobs layer. You write functions that
react to events or run on a cron; Inngest handles queueing, retries,
step memoization, and concurrency. No Redis or worker process to babysit.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add inngest
pnpm add -D inngest-cli
```

------------------------------------------------------------------------

## STEP 2 : Create The Client

```ts
// lib/inngest/client.ts
import { Inngest } from "inngest";

export const inngest = new Inngest({
  id: "my-app",
  // set in .env.local — never commit the key
  eventKey: process.env.INNGEST_EVENT_KEY,
});
```

```text
# .env.local
INNGEST_EVENT_KEY=placeholder_event_key
INNGEST_SIGNING_KEY=signkey-prod-placeholder
```

------------------------------------------------------------------------

## STEP 3 : Write An Event-Driven Function With Steps

Each `step.run` is memoized: if a later step throws, completed steps are
NOT re-executed on retry. Steps are the unit of durability.

```ts
// lib/inngest/functions/welcome.ts
import { inngest } from "../client";

export const welcomeEmail = inngest.createFunction(
  { id: "welcome-email", retries: 4 },
  { event: "user/signed-up" },
  async ({ event, step }) => {
    const user = await step.run("load-user", async () => {
      return db.user.findUnique({ where: { id: event.data.userId } });
    });

    // Durable sleep — survives restarts, no timer held in memory
    await step.sleep("wait-1-day", "1d");

    await step.run("send-email", async () => {
      await mailer.send({ to: user.email, template: "welcome" });
    });

    return { sent: true };
  },
);
```

------------------------------------------------------------------------

## STEP 4 : Add A Cron (Scheduled) Function

```ts
// lib/inngest/functions/nightly.ts
import { inngest } from "../client";

export const nightlyCleanup = inngest.createFunction(
  { id: "nightly-cleanup" },
  { cron: "TZ=UTC 0 3 * * *" }, // 03:00 UTC daily
  async ({ step }) => {
    await step.run("purge-expired", () => db.session.deleteExpired());
  },
);
```

------------------------------------------------------------------------

## STEP 5 : Expose The Serve Route (App Router)

```ts
// app/api/inngest/route.ts
import { serve } from "inngest/next";
import { inngest } from "@/lib/inngest/client";
import { welcomeEmail } from "@/lib/inngest/functions/welcome";
import { nightlyCleanup } from "@/lib/inngest/functions/nightly";

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [welcomeEmail, nightlyCleanup],
});
```

------------------------------------------------------------------------

## STEP 6 : Send Events From Your App

```ts
// app/api/signup/route.ts
import { inngest } from "@/lib/inngest/client";

export async function POST(req: Request) {
  const { userId } = await req.json();
  await inngest.send({ name: "user/signed-up", data: { userId } });
  return Response.json({ ok: true });
}
```

------------------------------------------------------------------------

## STEP 7 : Run The Dev Server

```bash
# terminal 1 — your Next.js app
pnpm dev

# terminal 2 — Inngest dev server + local dashboard
pnpm dlx inngest-cli@latest dev -u http://localhost:3000/api/inngest
```

Open http://localhost:8288 to inspect runs, replay, and view step timing.
On Windows use PowerShell or Git Bash; the `pnpm dlx` command is identical.

------------------------------------------------------------------------

## STEP 8 : Flow Overview

```text
   app code                 Inngest platform            serve route
  ----------               -----------------           ------------
  inngest.send(evt) ---->  [ event queue ]
                                 |
                                 v
                           match functions
                                 |  POST /api/inngest
                                 v
                           step.run  ->  memoized result
                           step.sleep -> durable timer
                           (auto retry w/ backoff on throw)
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Wrap every side effect in step.run so retries never duplicate work
✓ Use namespaced event names like "user/signed-up" for clarity
✓ Prefer step.sleep / step.sleepUntil over in-memory setTimeout
✓ Set explicit retries and concurrency limits per function
✓ Keep INNGEST_SIGNING_KEY server-only; never expose to the client
✓ Make steps idempotent — assume they may run more than once
✓ Use the local dev dashboard to replay failed runs before shipping
✓ Split functions into one-file-per-concern under lib/inngest/functions
```
