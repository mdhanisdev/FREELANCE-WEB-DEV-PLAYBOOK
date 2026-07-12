# BETTERSTACK SETUP [ UPTIME MONITORING ]
------------------------------------------------------------------------
Better Stack (formerly Better Uptime) is our default monitoring service:
uptime checks, on-call scheduling, incident management, and status pages.
This guide adds a health endpoint to a Next.js App Router app and wires
it to Better Stack monitors, using TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Add a Health Endpoint

```ts
// app/api/health/route.ts
export const dynamic = "force-dynamic";

export async function GET() {
  const checks = { db: true, cache: true }; // replace with real probes
  const healthy = Object.values(checks).every(Boolean);
  return Response.json(
    { status: healthy ? "ok" : "degraded", checks },
    { status: healthy ? 200 : 503 }
  );
}
```

```text
Monitor flow
  Better Stack ──GET /api/health──▶ app
        ▲                              │
        └────── 200 ok / 503 down ─────┘
             │
             └─ on failure → alert → on-call → incident
```
------------------------------------------------------------------------
## STEP 2 : Create the Monitor

In the Better Stack dashboard: Monitors → Create monitor → HTTP(S),
target `https://your-app.com/api/health`, check every 30s, expected
status 200, and enable "SSL certificate expiry" alerts.
------------------------------------------------------------------------
## STEP 3 : Automate via API (Optional)

```bash
# .env.local
BETTERSTACK_API_TOKEN=your_uptime_api_token
```

```ts
// scripts/create-monitor.ts
const res = await fetch("https://uptime.betterstack.com/api/v2/monitors", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.BETTERSTACK_API_TOKEN}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    monitor_type: "status",
    url: "https://your-app.com/api/health",
    check_frequency: 30,
    request_timeout: 10,
  }),
});
console.log(res.status, await res.json());
```

```bash
pnpm dlx tsx scripts/create-monitor.ts
```
------------------------------------------------------------------------
## STEP 4 : Send Heartbeats for Cron Jobs

Passive checks confirm a scheduled job actually ran.

```ts
// in your cron/worker after success
await fetch(`https://uptime.betterstack.com/api/v1/heartbeat/${process.env.HEARTBEAT_KEY}`);
```

If the heartbeat is not received within its grace period, Better Stack
raises an incident automatically.
------------------------------------------------------------------------
## STEP 5 : Configure Alerts & Status Page

Set escalation policies (email → SMS → phone), connect Slack, and publish
a public status page that reads directly from your monitors.
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# curl the endpoint
curl http://localhost:3000/api/health
```

Windows note: use `curl.exe` in PowerShell (bare `curl` is aliased to
Invoke-WebRequest). In Git Bash, `curl` works as written above.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Expose a real health endpoint that probes dependencies
✓ Return 503 (not 200) when a dependency is degraded
✓ Keep the health route dynamic so it is never cached
✓ Monitor SSL expiry alongside uptime
✓ Use heartbeats to catch silent cron failures
✓ Define escalation policies before an incident, not during
✓ Keep API tokens server-side; never commit them
✓ Publish a status page to cut inbound "is it down?" tickets
```
