# K6 SETUP [ LOAD TESTING ]

------------------------------------------------------------------------

Grafana k6 is a scriptable load-testing tool where scenarios are written
in plain JavaScript and executed by a high-performance Go engine. It is
the default choice for stress-testing Next.js API routes and server
actions before a release.

------------------------------------------------------------------------

## STEP 1 : Install The k6 Binary

k6 is a standalone binary, not an npm dependency.

```bash
# macOS
brew install k6

# Linux (Debian/Ubuntu)
sudo gpg -k && sudo apt-get install k6

# Windows
winget install k6 --source winget
# or: choco install k6
```

On Windows, run tests from PowerShell or Git Bash; the binary lands on
your PATH automatically after `winget`.

------------------------------------------------------------------------

## STEP 2 : Scaffold A Tests Folder (layout)

```text
   load-tests/
   ├── smoke.js        → 1 VU, sanity check
   ├── load.js         → expected traffic
   ├── stress.js       → find the breaking point
   └── lib/
       └── config.js   → shared thresholds + base URL
```

Keep load scripts out of `src/` so they never bundle into your app.

------------------------------------------------------------------------

## STEP 3 : Write A Load Scenario

```js
// load-tests/load.js
import http from "k6/http";
import { check, sleep } from "k6";

export const options = {
  stages: [
    { duration: "30s", target: 20 },  // ramp up
    { duration: "1m", target: 20 },   // steady state
    { duration: "30s", target: 0 },   // ramp down
  ],
  thresholds: {
    http_req_duration: ["p(95)<500"], // 95% under 500ms
    http_req_failed: ["rate<0.01"],   // <1% errors
  },
};

const BASE = __ENV.BASE_URL || "http://localhost:3000";

export default function () {
  const res = http.get(`${BASE}/api/health`);
  check(res, {
    "status is 200": (r) => r.status === 200,
    "body has ok": (r) => r.body.includes("ok"),
  });
  sleep(1);
}
```

------------------------------------------------------------------------

## STEP 4 : Provide A Health Route To Hit

```ts
// app/api/health/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({ status: "ok", ts: Date.now() });
}
```

------------------------------------------------------------------------

## STEP 5 : Run The Tests

```bash
# build + start production server first
pnpm build && pnpm start &

# smoke, then load
k6 run load-tests/smoke.js
k6 run -e BASE_URL=http://localhost:3000 load-tests/load.js
```

k6 prints a live summary; watch `http_req_duration` percentiles and the
checks pass rate.

------------------------------------------------------------------------

## STEP 6 : Read The Virtual-User Model

```text
   VUs (virtual users) each loop the default() function:

   VU-1  ── GET /api/health ── sleep(1) ── GET ── sleep(1) ─▶
   VU-2  ── GET /api/health ── sleep(1) ── GET ── sleep(1) ─▶
   ...
   VU-N  ─────────────────────────────────────────────────▶
          └── stages ramp N up and down over time ──┘
```

A threshold breach makes k6 exit non-zero, which fails your pipeline.

------------------------------------------------------------------------

## STEP 7 : Wire Into CI

```yaml
# .github/workflows/load.yml
name: load
on: workflow_dispatch
jobs:
  k6:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: grafana/setup-k6-action@v1
      - run: k6 run load-tests/load.js
```

Trigger load tests manually or on a schedule, never on every push.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Test against a production build, never the dev server
✓ Start every campaign with a 1-VU smoke test
✓ Define thresholds so failing runs exit non-zero and block CI
✓ Judge latency by p95/p99, not the average
✓ Parameterize BASE_URL with __ENV for reusable scripts
✓ Keep load-tests/ outside src/ so it never ships
✓ Ramp virtual users up and down — avoid instant spikes
✓ Run heavy stress tests against staging, not production
✓ Store trend results to catch regressions over releases
```
