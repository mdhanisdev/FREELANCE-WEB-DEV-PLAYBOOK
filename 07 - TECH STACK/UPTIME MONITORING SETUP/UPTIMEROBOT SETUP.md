# UPTIMEROBOT SETUP [ UPTIME MONITORING ]
------------------------------------------------------------------------
UptimeRobot is the alternative when you want simple, free uptime checks:
50 monitors on the free tier, 5-minute intervals, and a clean API. This
guide adds a health endpoint to Next.js App Router and registers a
monitor via API, using TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Add a Health Endpoint

```ts
// app/api/health/route.ts
export const dynamic = "force-dynamic";

export async function GET() {
  const healthy = true; // replace with real dependency probes
  return Response.json(
    { status: healthy ? "ok" : "down" },
    { status: healthy ? 200 : 503 }
  );
}
```

```text
Check loop
  UptimeRobot ─(every 5 min)─▶ GET /api/health ─▶ 200 = up
                                                 └ 5xx = down → alert
```
------------------------------------------------------------------------
## STEP 2 : Get an API Key

In the UptimeRobot dashboard: My Settings → API Settings → create a
"Main API Key". Store it locally.

```bash
# .env.local
UPTIMEROBOT_API_KEY=u123456-your_main_api_key
```
------------------------------------------------------------------------
## STEP 3 : Create a Monitor via API

```ts
// scripts/create-monitor.ts
const body = new URLSearchParams({
  api_key: process.env.UPTIMEROBOT_API_KEY!,
  format: "json",
  type: "1", // HTTP(S)
  url: "https://your-app.com/api/health",
  friendly_name: "App Health",
  interval: "300",
});

const res = await fetch("https://api.uptimerobot.com/v2/newMonitor", {
  method: "POST",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body,
});
console.log(await res.json());
```

```bash
pnpm dlx tsx scripts/create-monitor.ts
```
------------------------------------------------------------------------
## STEP 4 : Configure Alert Contacts

Add alert contacts (email, Slack webhook, or SMS) in the dashboard, then
attach them to the monitor so failures actually page someone.
------------------------------------------------------------------------
## STEP 5 : Add a Public Status Page

Create a Public Status Page from the dashboard and select your monitors;
UptimeRobot hosts it on a shareable URL with optional custom domain.
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
curl http://localhost:3000/api/health
```

Windows note: in PowerShell call `curl.exe`; the bare `curl` alias maps
to Invoke-WebRequest and formats output differently. Git Bash runs the
command as written.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Point monitors at a real health route, not the homepage
✓ Return 5xx when unhealthy so checks actually fail
✓ Keep the health route dynamic to avoid cached 200s
✓ Attach alert contacts before you rely on the monitor
✓ Use keyword monitors to catch soft failures (200 + error text)
✓ Keep the API key server-side; never ship it to the client
✓ Set intervals to match your SLA, not the minimum allowed
✓ Publish a status page to reduce inbound downtime questions
```
