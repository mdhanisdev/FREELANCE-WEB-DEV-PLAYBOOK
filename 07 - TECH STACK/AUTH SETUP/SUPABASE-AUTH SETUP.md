# SUPABASE AUTH SETUP [ SSR AUTH WITH POSTGRES + RLS ]
------------------------------------------------------------------------
Supabase Auth pairs a hosted Postgres with OAuth and magic-link login.
With @supabase/ssr the session lives in cookies and is refreshed by
middleware. Always call getUser() on the server for a verified identity,
and enforce access with Row Level Security. Never trust a role read from
the client.
------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add @supabase/supabase-js @supabase/ssr
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables

```text
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-placeholder
```

The anon key is safe to expose; RLS is what protects your data. Keep the
service role key server-only and never ship it to the browser.
------------------------------------------------------------------------
## STEP 3 : Server Client

```ts
// lib/supabase/server.ts
import { createServerClient } from "@supabase/ssr"
import { cookies } from "next/headers"

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (list) =>
          list.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)),
      },
    },
  )
}
```
------------------------------------------------------------------------
## STEP 4 : Middleware Session Refresh

```ts
// middleware.ts
import { createServerClient } from "@supabase/ssr"
import { NextResponse, type NextRequest } from "next/server"

export async function middleware(req: NextRequest) {
  const res = NextResponse.next({ request: req })
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => req.cookies.getAll(),
        setAll: (list) =>
          list.forEach(({ name, value, options }) =>
            res.cookies.set(name, value, options)),
      },
    },
  )
  await supabase.auth.getUser() // refreshes the session cookie
  return res
}

export const config = { matcher: ["/((?!_next|.*\\..*).*)"] }
```
------------------------------------------------------------------------
## STEP 5 : OAuth and Magic Link

```tsx
"use client"
import { createBrowserClient } from "@supabase/ssr"

const supabase = createBrowserClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
)

export function Login() {
  return (
    <>
      <button onClick={() => supabase.auth.signInWithOAuth({
        provider: "google",
        options: { redirectTo: `${location.origin}/auth/callback` },
      })}>Google</button>
      <button onClick={() => supabase.auth.signInWithOtp({
        email: "user@example.com",
        options: { emailRedirectTo: `${location.origin}/auth/callback` },
      })}>Magic link</button>
    </>
  )
}
```

Callback route that exchanges the code for a session:

```ts
// app/auth/callback/route.ts
import { createClient } from "@/lib/supabase/server"
import { NextResponse } from "next/server"

export async function GET(req: Request) {
  const code = new URL(req.url).searchParams.get("code")
  if (code) {
    const supabase = await createClient()
    await supabase.auth.exchangeCodeForSession(code)
  }
  return NextResponse.redirect(new URL("/dashboard", req.url))
}
```
------------------------------------------------------------------------
## STEP 6 : Verify on the Server with getUser

```tsx
// app/dashboard/page.tsx
import { createClient } from "@/lib/supabase/server"
import { redirect } from "next/navigation"

export default async function Dashboard() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser() // re-verified
  if (!user) redirect("/login")
  return <h1>Welcome {user.email}</h1>
}
```

Use `getUser()`, not `getSession()`, for authorization: getUser()
revalidates the token with the Auth server, while getSession() only
reads the (spoofable) cookie.
------------------------------------------------------------------------
## AUTH FLOW

```text
  Browser              Next.js Server         Supabase Auth
    | signInWithOAuth ----------------------------> provider consent
    | <------------------------------------------- ?code=...
    | GET /auth/callback --> exchangeCodeForSession -> issue tokens
    | <-- set cookies + 302 |                     |
    | request /dashboard    |                     |
    |---------------------->| getUser() ----------> verify JWT
    |                       | <------------------- verified user
    | <-- page or redirect -|                     |
    | (middleware refreshes the session on every request)
```
------------------------------------------------------------------------
## WHEN TO CHOOSE SUPABASE AUTH

```text
✓ You already use Supabase Postgres and want auth in the same stack
✓ You want Row Level Security to enforce access at the database
✓ You need magic links and OAuth without running an auth server
✓ You want realtime, storage, and auth from one provider
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Authorize with getUser() on the server; never trust getSession cookies
✓ Enable RLS on every table and write policies against auth.uid()
✓ Never expose the service role key to the browser
✓ Refresh the session in middleware so cookies stay valid
✓ Never trust a client-sent role; enforce it in RLS or server code
✓ Redirect OAuth and magic links only to your own origin
✓ Re-verify the user inside each server action and route handler
```
