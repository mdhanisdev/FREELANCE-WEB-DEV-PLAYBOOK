# AUTH.JS SETUP [ SESSION AUTH FOR NEXT.JS APP ROUTER ]
------------------------------------------------------------------------
Auth.js (NextAuth v5) is the batteries-included session library for
Next.js. It owns OAuth, credentials, JWT/session cookies, and a Prisma
adapter for persistence. Server is the source of truth: the session is
re-read and re-verified on every request. Never trust a role sent by the
browser.
------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add next-auth@beta @auth/prisma-adapter
pnpm add -D prisma
pnpm add @prisma/client
```

The `@beta` tag installs NextAuth v5, which is built for the App Router.
------------------------------------------------------------------------
## STEP 2 : Generate the Auth Secret

```bash
pnpm dlx auth secret
```

On Windows this writes `AUTH_SECRET` into `.env.local`. If you prefer to
do it manually:

```text
AUTH_SECRET=replace-with-a-32-byte-random-string
AUTH_GOOGLE_ID=your-google-client-id
AUTH_GOOGLE_SECRET=your-google-client-secret
AUTH_GITHUB_ID=your-github-client-id
AUTH_GITHUB_SECRET=your-github-client-secret
DATABASE_URL=postgresql://user@example.com/app
```

Never commit `.env.local`. Add it to `.gitignore`.
------------------------------------------------------------------------
## STEP 3 : Prisma Adapter Schema

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  role          String    @default("user")
  accounts      Account[]
  sessions      Session[]
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  provider          String
  providerAccountId String
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

```bash
pnpm dlx prisma migrate dev --name init-auth
```
------------------------------------------------------------------------
## STEP 4 : Auth Instance and Providers

```ts
// auth.ts
import NextAuth from "next-auth"
import { PrismaAdapter } from "@auth/prisma-adapter"
import Google from "next-auth/providers/google"
import GitHub from "next-auth/providers/github"
import { prisma } from "@/lib/prisma"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: "database" },
  providers: [Google, GitHub],
  callbacks: {
    session({ session, user }) {
      session.user.role = user.role // trusted: comes from DB, not client
      return session
    },
  },
})
```
------------------------------------------------------------------------
## STEP 5 : Route Handler

```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"
export const { GET, POST } = handlers
```
------------------------------------------------------------------------
## STEP 6 : Protect Server Components

```tsx
// app/dashboard/page.tsx
import { auth } from "@/auth"
import { redirect } from "next/navigation"

export default async function Dashboard() {
  const session = await auth()          // verified on the server
  if (!session?.user) redirect("/login")
  if (session.user.role !== "admin") redirect("/")
  return <h1>Welcome {session.user.email}</h1>
}
```
------------------------------------------------------------------------
## STEP 7 : Middleware Gate

```ts
// middleware.ts
export { auth as middleware } from "@/auth"
export const config = { matcher: ["/dashboard/:path*", "/admin/:path*"] }
```

Middleware is a coarse filter. Always re-check the session and role
inside the server component or route handler too.
------------------------------------------------------------------------
## STEP 8 : Sign In / Sign Out

```tsx
// components/auth-buttons.tsx
import { signIn, signOut } from "@/auth"

export function SignIn() {
  return (
    <form action={async () => { "use server"; await signIn("github") }}>
      <button type="submit">Sign in with GitHub</button>
    </form>
  )
}

export function SignOut() {
  return (
    <form action={async () => { "use server"; await signOut() }}>
      <button type="submit">Sign out</button>
    </form>
  )
}
```
------------------------------------------------------------------------
## AUTH FLOW

```text
  Browser            Next.js Server           OAuth Provider      DB
    |  click sign in      |                         |            |
    |-------------------->| redirect to provider    |            |
    |------------------------------------------------>|           |
    |  approve            |                         |            |
    |<------------------------------------------------|           |
    |  ?code=...          |                         |            |
    |-------------------->| exchange code + verify  |            |
    |                     |------------------------------------->| upsert user
    |                     | set httpOnly session cookie          |
    |<--------------------|                         |            |
    | request /dashboard  |                         |            |
    |-------------------->| auth() re-reads session from DB ---->| verify
    |<--- page (or 302) --|                         |            |
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Always call auth() on the server; never trust client role checks
✓ Store roles in the DB and inject them via the session callback
✓ Keep AUTH_SECRET out of git; rotate it if leaked
✓ Use session cookies marked httpOnly, secure, sameSite=lax
✓ Treat middleware as a gate, not the authorization decision
✓ Re-verify role at every mutation (server action / route handler)
✓ Use the database session strategy for instant revocation
✓ Pin next-auth@beta and test upgrades in a branch
```
