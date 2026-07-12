# MULTI-TENANCY SETUP [ MULTI-TENANCY ]

------------------------------------------------------------------------

The default multi-tenancy model is a shared database with an `orgId`
discriminator column. Every tenant's rows live in the same tables, scoped
by organization, with memberships mapping users to orgs and RBAC gating
actions. It is the simplest model to operate and scales to thousands of
tenants.

------------------------------------------------------------------------

## STEP 1 : Model Orgs, Memberships, And Roles

```ts
// db/schema.ts (Drizzle ORM)
import { pgTable, text, uuid, timestamp, pgEnum } from "drizzle-orm/pg-core";

export const roleEnum = pgEnum("role", ["owner", "admin", "member"]);

export const orgs = pgTable("orgs", {
  id: uuid("id").defaultRandom().primaryKey(),
  name: text("name").notNull(),
  slug: text("slug").notNull().unique(),
});

export const users = pgTable("users", {
  id: uuid("id").defaultRandom().primaryKey(),
  email: text("email").notNull().unique(),
});

export const memberships = pgTable("memberships", {
  id: uuid("id").defaultRandom().primaryKey(),
  orgId: uuid("org_id").notNull().references(() => orgs.id),
  userId: uuid("user_id").notNull().references(() => users.id),
  role: roleEnum("role").notNull().default("member"),
  createdAt: timestamp("created_at").defaultNow(),
});

// EVERY tenant-owned table carries org_id
export const projects = pgTable("projects", {
  id: uuid("id").defaultRandom().primaryKey(),
  orgId: uuid("org_id").notNull().references(() => orgs.id),
  name: text("name").notNull(),
});
```

------------------------------------------------------------------------

## STEP 2 : Visualize The Access Graph

```text
   User ──< Membership >── Org ──< Project, Invoice, ... (org_id)
              │
            role: owner | admin | member
                          │
                          ▼
              RBAC check gates every mutation

   Isolation rule: NO query touches a tenant table
   without a WHERE org_id = :activeOrg clause.
```

------------------------------------------------------------------------

## STEP 3 : Resolve The Active Org Per Request

```ts
// lib/tenant.ts
import { cookies } from "next/headers";
import { db } from "@/db";
import { memberships } from "@/db/schema";
import { and, eq } from "drizzle-orm";

export async function requireOrg(userId: string) {
  const cookieStore = await cookies();
  const orgId = cookieStore.get("active_org")?.value;
  if (!orgId) throw new Error("NO_ACTIVE_ORG");

  const [m] = await db
    .select()
    .from(memberships)
    .where(and(eq(memberships.userId, userId), eq(memberships.orgId, orgId)));

  if (!m) throw new Error("FORBIDDEN"); // user is not in this org
  return { orgId, role: m.role };
}
```

------------------------------------------------------------------------

## STEP 4 : Scope Every Query

```ts
// app/actions/projects.ts
"use server";
import { db } from "@/db";
import { projects } from "@/db/schema";
import { eq } from "drizzle-orm";
import { getSession } from "@/lib/auth";
import { requireOrg } from "@/lib/tenant";

export async function listProjects() {
  const { user } = await getSession();
  const { orgId } = await requireOrg(user.id);
  // org_id filter is mandatory — never optional
  return db.select().from(projects).where(eq(projects.orgId, orgId));
}
```

------------------------------------------------------------------------

## STEP 5 : Enforce RBAC

```ts
// lib/rbac.ts
type Role = "owner" | "admin" | "member";
const RANK: Record<Role, number> = { member: 1, admin: 2, owner: 3 };

export function can(role: Role, minimum: Role) {
  return RANK[role] >= RANK[minimum];
}

// usage inside a server action
const { role } = await requireOrg(user.id);
if (!can(role, "admin")) throw new Error("FORBIDDEN");
```

------------------------------------------------------------------------

## STEP 6 : Guard At The Edge

```ts
// middleware.ts
import { NextResponse, type NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const org = req.cookies.get("active_org");
  if (req.nextUrl.pathname.startsWith("/app") && !org) {
    return NextResponse.redirect(new URL("/select-org", req.url));
  }
  return NextResponse.next();
}

export const config = { matcher: ["/app/:path*"] };
```

For hard isolation guarantees, layer Postgres Row-Level Security so the
`org_id` filter is enforced by the database, not just app code.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Put org_id on every tenant-owned table and index it
✓ Never run a tenant query without a WHERE org_id clause
✓ Resolve active org server-side, never trust the client
✓ Verify membership before honoring any active_org cookie
✓ Centralize scoping in one helper so it can't be forgotten
✓ Model roles as ranks so RBAC checks stay declarative
✓ Add Postgres RLS as defense-in-depth behind app filters
✓ Cascade deletes by org for clean tenant offboarding
✓ Log the acting user + org on every mutation for audit
```
