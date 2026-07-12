# REDIS SETUP [ IN-MEMORY STORE / SERVERLESS CACHING ]
------------------------------------------------------------------------

Redis is a fast in-memory key/value store. For Next.js App Router on
serverless platforms, Upstash gives you Redis over HTTP — no persistent
socket, so it works from edge functions and scales to zero.

------------------------------------------------------------------------

## STEP 1 : Create An Upstash Database

- Sign up at Upstash and create a Redis database.
- Pick a region close to your app's deployment region.
- Copy the REST URL and REST token from the dashboard.

```text
   Next.js (App Router / Edge)
          |
     HTTPS (REST)
          |
   +--------------------+
   |  Upstash Redis     |
   |  global / regional |
   +--------------------+
```

------------------------------------------------------------------------

## STEP 2 : Store Credentials

Upstash exposes an HTTP REST endpoint plus a token. These work from any
runtime, including the edge, without a raw TCP connection.

```bash
# .env.local
UPSTASH_REDIS_REST_URL="https://us1-cool-name-12345.upstash.io"
UPSTASH_REDIS_REST_TOKEN="AX_______________placeholder_token_______________"
```

------------------------------------------------------------------------

## STEP 3 : Install The Client

```bash
pnpm add @upstash/redis
# optional rate limiting helper:
pnpm add @upstash/ratelimit
```

------------------------------------------------------------------------

## STEP 4 : Create The Client

The client is stateless (HTTP), so a module-level instance is fine —
`Redis.fromEnv()` reads the two env vars automatically.

```ts
// lib/redis.ts
import { Redis } from "@upstash/redis";

export const redis = Redis.fromEnv();
```

------------------------------------------------------------------------

## STEP 5 : Common Uses

### Cache (with TTL)

```ts
import { redis } from "@/lib/redis";

export async function getUser(id: string) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return cached;

  const user = await fetchUserFromDb(id);
  await redis.set(`user:${id}`, user, { ex: 60 }); // expire 60s
  return user;
}
```

### Sessions

```ts
await redis.set(`session:${token}`, { userId }, { ex: 60 * 60 * 24 });
const session = await redis.get(`session:${token}`);
```

### Rate Limiting

```ts
import { Ratelimit } from "@upstash/ratelimit";
import { redis } from "@/lib/redis";

const limiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, "10 s"),
});

const { success } = await limiter.limit(ip);
if (!success) return new Response("Too many requests", { status: 429 });
```

```text
+-----------------------------------------------+
| KEY                VALUE            TTL       |
| user:42            {json}           60s cache |
| session:abc        {userId}         24h auth  |
| ratelimit:1.2.3    counter          10s limit |
+-----------------------------------------------+
```

------------------------------------------------------------------------

## WHEN TO USE REDIS

- Caching expensive DB queries or API responses (with a TTL).
- Session and token storage that must be read on every request.
- Rate limiting and abuse protection.
- Ephemeral counters, feature flags, queues, leaderboards.

Redis is not your source of truth — it is memory. Treat data as
disposable and keep the durable copy in Postgres, MySQL, or MongoDB.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always set a TTL (ex) on cache and session keys
✓ Namespace keys with prefixes like user:, session:, ratelimit:
✓ Use the REST client so edge runtime works without sockets
✓ Never store the only copy of important data in Redis
✓ Keep REST token secret — it grants full database access
✓ Pick a region matching your app to cut latency
✓ Use @upstash/ratelimit instead of hand-rolling counters
✓ Invalidate or version cache keys when the source changes
```
