# DOCKER VPS SETUP [ HOSTING / SELF-HOSTED NEXT.JS ]

------------------------------------------------------------------------

Self-host Next.js on any VPS (Hetzner, DigitalOcean, AWS EC2) using a
multi-stage Docker image built from the `standalone` output. Maximum
control over cost, region, and runtime — you own the ops.

------------------------------------------------------------------------

## STEP 1 : Enable Standalone Output

`standalone` copies only the runtime files and a minimal server into
`.next/standalone`, producing a tiny final image.

```json
// next.config.ts (typed config)
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone",
};

export default nextConfig;
```

------------------------------------------------------------------------

## STEP 2 : Multi-Stage Dockerfile

Three stages: deps (install), builder (compile), runner (tiny runtime).

```dockerfile
# ---- deps ----
FROM node:20-alpine AS deps
WORKDIR /app
RUN corepack enable && corepack prepare pnpm@9 --activate
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# ---- builder ----
FROM node:20-alpine AS builder
WORKDIR /app
RUN corepack enable && corepack prepare pnpm@9 --activate
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm build

# ---- runner ----
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
ENV PORT=3000 HOSTNAME=0.0.0.0
CMD ["node", "server.js"]
```

------------------------------------------------------------------------

## STEP 3 : .dockerignore

Keep the build context small and secret-free.

```text
node_modules
.next
.git
.env*
npm-debug.log*
Dockerfile
.dockerignore
README.md
coverage
.vscode
```

------------------------------------------------------------------------

## STEP 4 : Build + Run

On Windows (PowerShell/bash), build and test locally with Docker
Desktop, then push to the VPS registry or build on the server.

```bash
docker build -t myapp:latest .
docker run -p 3000:3000 --env-file .env.production myapp:latest
```

```text
   HOST :3000  ──>  container :3000  ──>  node server.js
                                          (.next/standalone)
```

------------------------------------------------------------------------

## STEP 5 : Deploy On The VPS

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "3000:3000"
    env_file: .env.production
    restart: unless-stopped
```

```bash
# on the server
docker compose up -d --build
docker compose logs -f web
```

Put a reverse proxy (Caddy or nginx) in front for TLS and to route
port 443 -> 3000.

```text
   Internet ──> :443 Caddy/nginx (TLS) ──> :3000 Next.js container
                    auto Let's Encrypt        docker compose
```

------------------------------------------------------------------------

## WHEN TO SELF-HOST

```text
✓ Need a fixed monthly cost / predictable billing
✓ Data-residency or compliance requires a specific region
✓ Want full control of the runtime and OS
✓ Already run other services on the same box
✗ You want zero ops — use Vercel/Netlify/Railway instead
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always set output: "standalone" for slim images
✓ Use multi-stage builds; never ship node_modules to runner
✓ Run as a non-root user (nextjs) inside the container
✓ Keep .env* out of the image via .dockerignore
✓ Pin base image + pnpm versions for reproducible builds
✓ Terminate TLS at a reverse proxy, not in Node
✓ Use restart: unless-stopped so the app survives reboots
✓ Ship logs to a collector; never grep containers in prod
```
