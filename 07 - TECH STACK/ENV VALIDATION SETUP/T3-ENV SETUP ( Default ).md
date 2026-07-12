# T3-ENV SETUP [ ENV VALIDATION / TYPE-SAFE CONFIG ]
------------------------------------------------------------------------
@t3-oss/env-nextjs validates environment variables at build and boot
time using Zod. It fails fast on missing or malformed variables and
gives fully typed, autocompleted access to `env` across the app.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add @t3-oss/env-nextjs zod
```

Both are runtime dependencies because `env` is imported by app code,
not just tooling.
------------------------------------------------------------------------
## STEP 2 : Create src/env.ts

```ts
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    NODE_ENV: z
      .enum(["development", "test", "production"])
      .default("development"),
    AUTH_SECRET: z.string().min(32),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    NODE_ENV: process.env.NODE_ENV,
    AUTH_SECRET: process.env.AUTH_SECRET,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
  emptyStringAsUndefined: true,
});
```

Client variables MUST be prefixed `NEXT_PUBLIC_`; the library enforces
this and will error otherwise.
------------------------------------------------------------------------
## STEP 3 : Validate During next.config

Importing `env` in `next.config.ts` runs validation at build start:

```ts
import type { NextConfig } from "next";
import "./src/env";

const nextConfig: NextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

A bad or missing variable now aborts the build with a readable error
instead of failing mysteriously at runtime.
------------------------------------------------------------------------
## STEP 4 : Create .env.example

Commit a template; never commit real secrets:

```text
# Server
DATABASE_URL="postgresql://user:pass@localhost:5432/app"
AUTH_SECRET="replace-with-32-char-min-random-string"
NODE_ENV="development"

# Client
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

On Windows, copy it with:

```bash
cp .env.example .env
```
------------------------------------------------------------------------
## STEP 5 : Usage in Code

```ts
import { env } from "@/env";

// Server component / route handler
const db = connect(env.DATABASE_URL);

// Client component
const url = env.NEXT_PUBLIC_APP_URL;
```

```text
   .env / process.env
          |
          v
     src/env.ts  --- Zod schema --->  validate
          |                              |
     valid | invalid                     |
          |     \                        |
          v      \--> throw (build fails)
   typed env object
     |            \
  server keys   NEXT_PUBLIC_* client keys
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Import "./src/env" in next.config to validate at build time
✓ Prefix every browser variable with NEXT_PUBLIC_
✓ Keep server secrets out of the client object entirely
✓ Set emptyStringAsUndefined so blank vars trigger defaults
✓ Commit .env.example, never commit the real .env
✓ Import env from "@/env" instead of raw process.env
✓ Use z.string().url() and min lengths to catch typos early
✓ Add .env* to .gitignore except .env.example
```
