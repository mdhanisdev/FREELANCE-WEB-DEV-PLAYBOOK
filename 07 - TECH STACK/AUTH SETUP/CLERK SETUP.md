# CLERK SETUP [ HOSTED AUTH WITH PREBUILT UI + ORGS ]
------------------------------------------------------------------------
Clerk is a fully hosted auth provider with drop-in React components,
managed sessions, and first-class organizations/multi-tenancy. You ship
sign-in in minutes. Sessions live in Clerk; your server verifies them on
every request via `auth()`. Never trust a role read in the browser.
------------------------------------------------------------------------
## STEP 1 : Install

```bash
pnpm add @clerk/nextjs
```

Create the app at the Clerk dashboard and copy the two keys.
------------------------------------------------------------------------
## STEP 2 : Environment Variables

```text
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_placeholder
CLERK_SECRET_KEY=sk_test_placeholder
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
```

Only the publishable key is exposed to the browser. Keep the secret key
server-side and out of git.
------------------------------------------------------------------------
## STEP 3 : ClerkProvider in the Root Layout

```tsx
// app/layout.tsx
import { ClerkProvider } from "@clerk/nextjs"

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```
------------------------------------------------------------------------
## STEP 4 : Middleware

```ts
// middleware.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server"

const isProtected = createRouteMatcher(["/dashboard(.*)", "/admin(.*)"])

export default clerkMiddleware(async (auth, req) => {
  if (isProtected(req)) await auth.protect()
})

export const config = {
  matcher: ["/((?!_next|.*\\..*).*)", "/(api|trpc)(.*)"],
}
```
------------------------------------------------------------------------
## STEP 5 : Sign-In / Sign-Up Pages

```tsx
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from "@clerk/nextjs"
export default function Page() { return <SignIn /> }
```

```tsx
// app/sign-up/[[...sign-up]]/page.tsx
import { SignUp } from "@clerk/nextjs"
export default function Page() { return <SignUp /> }
```

Header widgets:

```tsx
import { UserButton, SignedIn, SignedOut, SignInButton } from "@clerk/nextjs"

export function Nav() {
  return (
    <nav>
      <SignedOut><SignInButton /></SignedOut>
      <SignedIn><UserButton /></SignedIn>
    </nav>
  )
}
```
------------------------------------------------------------------------
## STEP 6 : Protect Server Components and Routes

```tsx
// app/dashboard/page.tsx
import { auth } from "@clerk/nextjs/server"
import { redirect } from "next/navigation"

export default async function Dashboard() {
  const { userId, sessionClaims } = await auth() // verified server-side
  if (!userId) redirect("/sign-in")
  const role = sessionClaims?.metadata?.role     // from Clerk, not client
  if (role !== "admin") redirect("/")
  return <h1>Admin dashboard</h1>
}
```

```ts
// app/api/secret/route.ts
import { auth } from "@clerk/nextjs/server"

export async function GET() {
  const { userId } = await auth()
  if (!userId) return new Response("Unauthorized", { status: 401 })
  return Response.json({ ok: true })
}
```
------------------------------------------------------------------------
## STEP 7 : Organizations (Multi-Tenant)

```tsx
import { OrganizationSwitcher } from "@clerk/nextjs"
// in a server component:
import { auth } from "@clerk/nextjs/server"
const { orgId, orgRole } = await auth()
```

Enable Organizations in the dashboard to get orgs, roles, and invites
without building tables yourself.
------------------------------------------------------------------------
## AUTH FLOW

```text
  Browser              Next.js Server            Clerk API
    | open /sign-in         |                        |
    | <SignIn/> renders     |                        |
    | submit credentials -----------------------------> verify + issue session
    | <----------------------------------------------- session token (cookie)
    | request /dashboard    |                        |
    |---------------------->| auth() ----------------> verify token + claims
    |                       | <---------------------- userId, role, orgId
    | <-- page or redirect -|                        |
```
------------------------------------------------------------------------
## WHEN TO CHOOSE CLERK

```text
✓ You want production auth today with zero UI to build
✓ You need organizations / B2B multi-tenancy out of the box
✓ You are happy with a hosted provider and its pricing
✓ You want managed MFA, device sessions, and user management UI
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Verify identity server-side with auth(); never trust client role
✓ Store roles in Clerk public/private metadata, read from sessionClaims
✓ Keep CLERK_SECRET_KEY server-only and out of version control
✓ Use auth.protect() in middleware plus a re-check in the handler
✓ Return 401 from API routes when userId is missing
✓ Scope every query by orgId in multi-tenant apps
✓ Confirm role again inside each mutation, not just on page load
```
