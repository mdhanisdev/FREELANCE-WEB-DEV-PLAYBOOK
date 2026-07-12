# RATE LIMITING SETUP [ SLIDING WINDOW WITH UPSTASH ]
------------------------------------------------------------------------
## STEP 1 : Why Rate Limit

Rate limiting protects endpoints from abuse: credential stuffing on
login, spam on signup, and cost blowups on public APIs. Upstash
Ratelimit stores counters in Redis, so limits are shared across every
serverless instance.

```text
   Client (IP 203.0.113.5)
        |
        v
+---------------------+   over limit   +-------------------+
|  Ratelimit.limit()  |--------------->|  429 Too Many Req |
|  (Redis counters)   |                +-------------------+
+---------------------+   allowed
        |
        v
   Handler runs
```
------------------------------------------------------------------------
## STEP 2 : Install

```bash
pnpm add @upstash/ratelimit @upstash/redis
```

Uses the same `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN`
env vars as the Redis cache setup.
------------------------------------------------------------------------
## STEP 3 : Create a Limiter

Sliding window smooths bursts better than a fixed window. Enable
`analytics` to see usage in the Upstash console.

```ts
// lib/ratelimit.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const redis = Redis.fromEnv();

export const authLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(5, "60 s"),
  analytics: true,
  prefix: "rl:auth",
});

export const apiLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(30, "10 s"),
  analytics: true,
  prefix: "rl:api",
});
```

```text
Ratelimit.slidingWindow(5, "60 s")
  -> at most 5 requests in any rolling 60-second window per key
```
------------------------------------------------------------------------
## STEP 4 : Resolve the Client IP

Behind a proxy/CDN, read the forwarded header. Fall back to a constant so
a missing IP does not bypass the limit.

```ts
// lib/get-ip.ts
import { NextRequest } from "next/server";

export function getIp(req: NextRequest): string {
  const forwarded = req.headers.get("x-forwarded-for");
  if (forwarded) return forwarded.split(",")[0].trim();
  return req.headers.get("x-real-ip") ?? "127.0.0.1";
}
```
------------------------------------------------------------------------
## STEP 5 : Apply in a Route Handler and Return 429

Call `limit()` with the IP key. When blocked, return HTTP 429 with the
standard rate-limit headers so clients can back off.

```ts
// app/api/login/route.ts
import { NextRequest, NextResponse } from "next/server";
import { authLimiter } from "@/lib/ratelimit";
import { getIp } from "@/lib/get-ip";

export async function POST(req: NextRequest) {
  const ip = getIp(req);
  const { success, limit, remaining, reset } = await authLimiter.limit(ip);

  if (!success) {
    return NextResponse.json(
      { error: "Too many requests. Try again later." },
      {
        status: 429,
        headers: {
          "X-RateLimit-Limit": limit.toString(),
          "X-RateLimit-Remaining": remaining.toString(),
          "X-RateLimit-Reset": reset.toString(),
          "Retry-After": Math.ceil((reset - Date.now()) / 1000).toString(),
        },
      }
    );
  }

  // ...verify credentials
  return NextResponse.json({ ok: true });
}
```
------------------------------------------------------------------------
## STEP 6 : What to Protect

```text
POST /api/login    -> 5 / 60s   (block credential stuffing)
POST /api/signup   -> 3 / 60s   (block spam accounts)
POST /api/contact  -> 3 / 60s   (block form spam)
GET  /api/public/* -> 30 / 10s  (fair-use throttle)
```

For authenticated routes, key by user ID instead of IP so a shared
office IP does not throttle everyone: `authLimiter.limit(userId)`.
------------------------------------------------------------------------
## STEP 7 : Optional — Limit in Middleware

To protect many routes at once, run the limiter in `middleware.ts`. Keep
it on paths where a Redis round-trip per request is acceptable.

```ts
import { NextRequest, NextResponse } from "next/server";
import { apiLimiter } from "@/lib/ratelimit";
import { getIp } from "@/lib/get-ip";

export async function middleware(req: NextRequest) {
  const { success } = await apiLimiter.limit(getIp(req));
  if (!success) {
    return NextResponse.json({ error: "Rate limited" }, { status: 429 });
  }
  return NextResponse.next();
}

export const config = { matcher: "/api/public/:path*" };
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Prefer slidingWindow over fixedWindow to avoid burst edges
✓ Key by IP for anonymous, by userId for authenticated routes
✓ Return 429 with Retry-After and X-RateLimit-* headers
✓ Use tight limits on login/signup/contact, looser on read APIs
✓ Never let a missing IP silently bypass the limiter
✓ Namespace each limiter with a distinct prefix
✓ Enable analytics to tune thresholds from real traffic
✓ Rate limit at the edge/middleware for broad coverage
```
