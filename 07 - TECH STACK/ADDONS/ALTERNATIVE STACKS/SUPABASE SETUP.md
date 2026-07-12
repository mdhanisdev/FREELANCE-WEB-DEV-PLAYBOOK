# SUPABASE SETUP [ ALTERNATIVE STACKS ]

------------------------------------------------------------------------

Supabase is an all-in-one backend: a managed Postgres database, Auth,
Storage, and Realtime behind one SDK. It replaces the DB + auth + file +
websocket layers of the default stack, with Row-Level Security (RLS) as
the authorization backbone.

------------------------------------------------------------------------

## STEP 1 : Install Client And CLI

```bash
pnpm add @supabase/supabase-js @supabase/ssr
pnpm add -D supabase
```

```bash
pnpm supabase init      # creates supabase/ with local config
pnpm supabase start     # local stack via Docker
```

On Windows, Supabase local dev requires Docker Desktop with WSL2; the CLI
binary itself runs fine from Git Bash or PowerShell.

------------------------------------------------------------------------

## STEP 2 : Configure Env

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=server-only-secret
```

Never expose the service-role key to the client — it bypasses RLS.

------------------------------------------------------------------------

## STEP 3 : Create Server + Browser Clients

```ts
// lib/supabase/server.ts
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

export async function createClient() {
  const cookieStore = await cookies();
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (all) =>
          all.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          ),
      },
    }
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Enforce RLS Policies

```sql
-- migrations: rows are private to their owner
alter table projects enable row level security;

create policy "owner can read" on projects
  for select using (auth.uid() = user_id);

create policy "owner can write" on projects
  for insert with check (auth.uid() = user_id);
```

```text
   Request ──▶ Supabase API ──▶ Postgres
                                  │
                          RLS policy runs with
                          auth.uid() from the JWT
                                  │
                     ┌────────────┴────────────┐
                   allow                      deny
```

RLS is the security boundary — the anon key is safe to ship precisely
because policies gate every row.

------------------------------------------------------------------------

## STEP 5 : Auth, Storage, Realtime

```ts
// auth
await supabase.auth.signInWithOtp({ email });

// storage upload
await supabase.storage.from("avatars").upload(`u/${id}.png`, file);

// realtime subscription
supabase
  .channel("projects")
  .on("postgres_changes", { event: "*", schema: "public", table: "projects" },
    (payload) => console.log(payload))
  .subscribe();
```

------------------------------------------------------------------------

## STEP 6 : Migrate And Deploy

```bash
pnpm supabase db diff -f add_projects   # generate migration
pnpm supabase db push                   # apply to linked project
pnpm supabase gen types typescript --linked > lib/database.types.ts
```

Generated types give you end-to-end type safety from table to component.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Enable RLS on every table — treat it as the auth layer
✓ Keep the service-role key server-only, never in the client
✓ Use @supabase/ssr for cookie-based auth in the App Router
✓ Generate TypeScript types from the schema after each migration
✓ Version schema changes as migrations, not dashboard edits
✓ Scope Realtime channels and secure them with RLS too
✓ Set Storage bucket policies; public buckets leak files
✓ Test policies locally with supabase start before pushing
✓ Prefer Supabase when you want DB + auth + files in one SDK
```
