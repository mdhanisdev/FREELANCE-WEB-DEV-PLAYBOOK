# UPSTASH REDIS SETUP [ SERVERLESS CACHE-ASIDE ]
------------------------------------------------------------------------
## STEP 1 : Why Upstash Redis

Upstash is a serverless, HTTP-based Redis that works from Edge and
serverless functions without connection pooling headaches. It is ideal
for a cache-aside layer in front of a slow database or third-party API.

```text
   Request
      |
      v
+-----------+   miss   +-------------+
|  Redis    |--------->|  Database   |
|  (cache)  |<---------|  (source)   |
+-----------+   fill   +-------------+
      | hit
      v
   Response (fast)
```
------------------------------------------------------------------------
## STEP 2 : Install

```bash
pnpm add @upstash/redis
```

Create a database at console.upstash.com and copy the REST credentials.
------------------------------------------------------------------------
## STEP 3 : Environment Variables

Add to `.env.local` (never commit it). `Redis.fromEnv()` reads these two
names automatically.

```text
UPSTASH_REDIS_REST_URL=https://your-db.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-token
```

On Windows, keep the file at the project root and ensure it is listed in
`.gitignore`.
------------------------------------------------------------------------
## STEP 4 : Create the Client

Instantiate once and reuse. `fromEnv` avoids hardcoding secrets.

```ts
// lib/redis.ts
import { Redis } from "@upstash/redis";

export const redis = Redis.fromEnv();
```
------------------------------------------------------------------------
## STEP 5 : Key Namespacing

Prefix keys by domain and version so you can invalidate groups and avoid
collisions. Use a helper to enforce the convention.

```ts
// lib/cache-keys.ts
const VERSION = "v1";

export const keys = {
  user: (id: string) => `${VERSION}:user:${id}`,
  post: (slug: string) => `${VERSION}:post:${slug}`,
  feed: (page: number) => `${VERSION}:feed:page:${page}`,
};
```

```text
v1:user:42          -> single user object
v1:post:hello-world -> rendered post payload
v1:feed:page:1      -> paginated list
```

Bump `VERSION` to invalidate everything at once during a schema change.
------------------------------------------------------------------------
## STEP 6 : Cache-Aside with TTL

Read from cache; on miss, load from source, write back with an
expiry, and return. Upstash serializes/deserializes JSON for you.

```ts
import { redis } from "@/lib/redis";
import { keys } from "@/lib/cache-keys";

const TTL_SECONDS = 60 * 5; // 5 minutes

export async function getPost(slug: string) {
  const key = keys.post(slug);

  const cached = await redis.get<Post>(key);
  if (cached) return cached;

  const post = await db.post.findUnique({ where: { slug } });
  if (post) {
    await redis.set(key, post, { ex: TTL_SECONDS });
  }
  return post;
}
```
------------------------------------------------------------------------
## STEP 7 : Invalidation on Write

Delete or overwrite the key whenever the source changes, so stale data
never lingers past a mutation.

```ts
export async function updatePost(slug: string, data: PostInput) {
  const post = await db.post.update({ where: { slug }, data });
  await redis.set(keys.post(slug), post, { ex: TTL_SECONDS });
  await redis.del(keys.feed(1)); // bust dependent lists
  return post;
}
```
------------------------------------------------------------------------
## STEP 8 : Use in a Route Handler

```ts
// app/api/posts/[slug]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { getPost } from "@/lib/posts";

export async function GET(
  _req: NextRequest,
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) return NextResponse.json({ error: "Not found" }, { status: 404 });
  return NextResponse.json(post);
}
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use Redis.fromEnv() and keep tokens out of source control
✓ Instantiate one client module and import it everywhere
✓ Namespace keys as version:domain:id for clean invalidation
✓ Always set a TTL (ex) so stale data self-expires
✓ Invalidate on write: overwrite the key, del dependent lists
✓ Store only serializable JSON; keep payloads small
✓ Bump the version prefix to flush the whole cache on schema change
✓ Treat the cache as disposable — never the source of truth
```
