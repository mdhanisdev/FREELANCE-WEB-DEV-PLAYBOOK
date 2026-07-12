# BETTER-AUTH SETUP [ SELF-HOSTED TS-FIRST AUTH ]
------------------------------------------------------------------------
Better Auth is a framework-agnostic, TypeScript-first auth library you
own and self-host. It ships email/password, social login, and sessions
with a typed server API and client. Your database is the source of
truth; every request re-validates the session on the server. Never
trust a role sent by the browser.
------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add better-auth
pnpm add @prisma/client
pnpm add -D prisma
```
------------------------------------------------------------------------
## STEP 2 : Environment Variables

```text
BETTER_AUTH_SECRET=replace-with-a-32-byte-random-string
BETTER_AUTH_URL=http://localhost:3000
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
DATABASE_URL=postgresql://user@example.com/app
```
------------------------------------------------------------------------
## STEP 3 : Auth Instance

```ts
// lib/auth.ts
import { betterAuth } from "better-auth"
import { prismaAdapter } from "better-auth/adapters/prisma"
import { prisma } from "@/lib/prisma"

export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
  socialProviders: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
  },
})
```

Generate the schema and migrate:

```bash
pnpm dlx @better-auth/cli generate
pnpm dlx prisma migrate dev --name init-auth
```
------------------------------------------------------------------------
## STEP 4 : Route Handler

```ts
// app/api/auth/[...all]/route.ts
import { auth } from "@/lib/auth"
import { toNextJsHandler } from "better-auth/next-js"

export const { GET, POST } = toNextJsHandler(auth)
```
------------------------------------------------------------------------
## STEP 5 : Client

```ts
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL,
})

export const { signIn, signUp, signOut, useSession } = authClient
```

Email/password and social sign-in from a client component:

```tsx
"use client"
import { signIn, signUp } from "@/lib/auth-client"

export function Login() {
  return (
    <>
      <button onClick={() => signIn.email({ email: "user@example.com", password: "secret" })}>
        Sign in
      </button>
      <button onClick={() => signIn.social({ provider: "github" })}>
        GitHub
      </button>
    </>
  )
}
```
------------------------------------------------------------------------
## STEP 6 : Read the Session on the Server

```tsx
// app/dashboard/page.tsx
import { auth } from "@/lib/auth"
import { headers } from "next/headers"
import { redirect } from "next/navigation"

export default async function Dashboard() {
  const session = await auth.api.getSession({ headers: await headers() })
  if (!session) redirect("/login")
  if (session.user.role !== "admin") redirect("/") // role from DB, trusted
  return <h1>Welcome {session.user.email}</h1>
}
```
------------------------------------------------------------------------
## STEP 7 : Protect API Routes

```ts
// app/api/secret/route.ts
import { auth } from "@/lib/auth"
import { headers } from "next/headers"

export async function GET() {
  const session = await auth.api.getSession({ headers: await headers() })
  if (!session) return new Response("Unauthorized", { status: 401 })
  return Response.json({ ok: true })
}
```
------------------------------------------------------------------------
## AUTH FLOW

```text
  Browser              Next.js Server            DB
    | signIn.email() ------> /api/auth/... |
    |                        | verify password hash ---> lookup user
    |                        | create session row ------> insert
    | <-- httpOnly cookie ---|                    |
    | request /dashboard     |                    |
    |----------------------->| getSession(headers) ----> validate session
    |                        | <----------------------- user + role
    | <-- page or redirect --|                    |
```
------------------------------------------------------------------------
## WHEN TO CHOOSE BETTER-AUTH

```text
✓ You want to fully own and self-host the auth stack
✓ You value end-to-end TypeScript types on server and client
✓ You use a framework other than Next.js, or many frameworks
✓ You want plugins (2FA, orgs, passkeys) without a hosted vendor
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Always resolve the session with auth.api.getSession on the server
✓ Never trust a role from the client; read it from the session/DB
✓ Keep BETTER_AUTH_SECRET out of git and rotate on leak
✓ Serve over HTTPS so session cookies stay secure
✓ Re-check authorization inside every mutation and API route
✓ Run the CLI generator after config changes to sync the schema
✓ Return 401 early when no session is present
✓ Enable rate limiting on email/password endpoints
```
