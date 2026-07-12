# NEXT.JS SETUP [ FULL-STACK REACT FRAMEWORK ]
------------------------------------------------------------------------
## STEP 1 : Scaffold The Project

Create a new App Router project with TypeScript and Tailwind preselected.

```bash
pnpm create next-app@latest my-app --ts --app --tailwind --eslint --src-dir --import-alias "@/*"
cd my-app
pnpm dev
```

Flags used: `--src-dir` keeps source under `src/`, `--import-alias "@/*"`
maps `@/` to `src/`. Windows note: run in PowerShell or Git Bash; paths
with spaces must be quoted (e.g. `cd "my app"`).
------------------------------------------------------------------------
## STEP 2 : Enforce Strict TypeScript

Open `tsconfig.json` and confirm strict mode plus useful guards.

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "bundler",
    "paths": { "@/*": ["./src/*"] }
  }
}
```

`noUncheckedIndexedAccess` catches out-of-bounds reads at compile time and
is the single highest-value flag for production safety.
------------------------------------------------------------------------
## STEP 3 : Recommended Folder Structure

Keep server-only code isolated from client components.

```text
src/
├── app/                 # routes, layouts, route handlers
│   ├── layout.tsx
│   ├── page.tsx
│   └── (marketing)/     # route groups do not affect the URL
├── components/          # reusable client/server components
│   └── ui/              # primitives (button, input, ...)
├── lib/                 # framework-agnostic utilities (cn, fetchers)
├── server/              # server-only: db, auth, actions
│   ├── db.ts
│   └── actions.ts
└── styles/              # globals.css, tokens
```

Mark server-only modules with `import "server-only"` at the top so they
throw if ever imported into a client bundle.
------------------------------------------------------------------------
## STEP 4 : Add The cn() Helper

Merge Tailwind classes without conflicts. Install the two dependencies.

```bash
pnpm add clsx tailwind-merge
```

```ts
// src/lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Usage: `className={cn("px-4 py-2", isActive && "bg-black text-white")}`.
------------------------------------------------------------------------
## STEP 5 : Base Scripts & Env

Standardize scripts in `package.json`.

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  }
}
```

Store secrets in `.env.local` (git-ignored). Never commit real values.

```text
DATABASE_URL="postgresql://user@example.com:5432/app"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

Only `NEXT_PUBLIC_*` variables reach the browser; everything else is
server-only. Access them via `process.env.DATABASE_URL` in server code.
------------------------------------------------------------------------
## STEP 6 : Request Flow Overview

```text
Browser request
   │
   ▼
app/layout.tsx  ──►  app/page.tsx (Server Component)
   │                     │
   │                     ├─ fetch() with caching / revalidate
   │                     └─ renders HTML on the server (RSC)
   ▼
Client Components ("use client")  ──►  hydrate + interactivity
   │
   └─ Server Actions / Route Handlers (app/api/*) for mutations
```
------------------------------------------------------------------------
## STEP 7 : Verify Production Build

Always confirm the app builds before deploying.

```bash
pnpm build
pnpm start
```

Fix any type or lint errors surfaced here; `next build` fails hard on
type errors when strict mode is on, which is the desired behavior.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Default to Server Components; add "use client" only where needed
✓ Isolate secrets in src/server and guard with import "server-only"
✓ Keep the cn() helper as the single source for class merging
✓ Enable noUncheckedIndexedAccess and treat build errors as blockers
✓ Use route groups (folder) to organize without changing URLs
✓ Prefer Server Actions over ad-hoc API routes for form mutations
✓ Run pnpm typecheck in CI, not just next lint
✓ Never expose secrets via NEXT_PUBLIC_* variables
```
