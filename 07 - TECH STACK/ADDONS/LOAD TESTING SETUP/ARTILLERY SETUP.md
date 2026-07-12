# ARTILLERY SETUP [ LOAD TESTING ]

------------------------------------------------------------------------

Artillery is a YAML-first load-testing toolkit that installs from npm and
excels at multi-step user flows and HTTP/WebSocket scenarios. Use it when
you prefer declarative configs over scripting, or need scenario weights.

------------------------------------------------------------------------

## STEP 1 : Install Artillery

```bash
pnpm add -D artillery
```

Being an npm package, Artillery needs no separate binary — handy on
Windows where installing native tools can be friction.

------------------------------------------------------------------------

## STEP 2 : Define A Scenario File

```yaml
# load-tests/artillery.yml
config:
  target: "http://localhost:3000"
  phases:
    - duration: 30
      arrivalRate: 5
      name: warm-up
    - duration: 60
      arrivalRate: 20
      name: sustained
  ensure:
    p95: 500
    maxErrorRate: 1

scenarios:
  - name: browse-and-fetch
    weight: 1
    flow:
      - get:
          url: "/api/health"
          expect:
            - statusCode: 200
      - think: 1
      - get:
          url: "/api/products"
          capture:
            - json: "$[0].id"
              as: "productId"
      - get:
          url: "/api/products/{{ productId }}"
```

------------------------------------------------------------------------

## STEP 3 : Add Scripts

```json
{
  "scripts": {
    "load": "artillery run load-tests/artillery.yml",
    "load:report": "artillery run --output report.json load-tests/artillery.yml"
  }
}
```

------------------------------------------------------------------------

## STEP 4 : Run And Report

```bash
pnpm build && pnpm start &
pnpm load
pnpm load:report && artillery report report.json
```

The `report` command turns `report.json` into a browsable HTML dashboard.

------------------------------------------------------------------------

## STEP 5 : Understand The Arrival Model

```text
   Artillery is ARRIVAL-based (users per second), not fixed-VU:

   phase: warm-up      phase: sustained
   5 users/s           20 users/s
   ▁▁▁▁▁               ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇
   each arrival runs the full scenario flow top-to-bottom:
     GET /health → think → GET /products → capture id → GET /products/:id
```

This differs from k6's virtual-user loop — Artillery injects new arrivals
at a rate, closer to real traffic bursts.

------------------------------------------------------------------------

## STEP 6 : Gate In CI

```yaml
# .github/workflows/artillery.yml
name: artillery
on: workflow_dispatch
jobs:
  load:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm load
```

The `ensure` block makes Artillery exit non-zero when p95 or error rate
breaches your budget, failing the job automatically.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Model arrival rate, not raw VUs, to mimic real bursts
✓ Use ensure thresholds so breaches fail CI automatically
✓ Chain requests with capture to test real user journeys
✓ Add think time between steps to simulate human pacing
✓ Weight multiple scenarios to reflect true traffic mix
✓ Always target a production build, not next dev
✓ Generate HTML reports to share results with the team
✓ Keep YAML in load-tests/ out of the app bundle
✓ Prefer Artillery over k6 when you want config over code
```
