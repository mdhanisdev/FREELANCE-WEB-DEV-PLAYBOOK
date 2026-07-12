# ROUTE HANDLERS SETUP [ NEXT.JS APP ROUTER API LAYER ]
------------------------------------------------------------------------

Route Handlers are the App Router replacement for `pages/api`. Each file
named `route.ts` inside `app/api/**` exports HTTP verb functions
(`GET`, `POST`, `PATCH`, `DELETE`) that run on the server only.

THE GOLDEN RULE (memorize this order for every mutation):

```text
authenticate  →  validate  →  authorize  →  do work  →  typed response
   (who?)        (shape?)     (allowed?)    (logic)     (ok / err)
```

Skip a step and you ship a bug. This doc wires all five.

------------------------------------------------------------------------
## STEP 1 : Install & Shared Helpers

```bash
pnpm add zod
```

Create one place for consistent responses so every handler looks alike.

```ts
// lib/http.ts
import { NextResponse } from "next/server";

export function ok<T>(data: T, init?: ResponseInit) {
  return NextResponse.json({ ok: true, data }, init);
}

export function err(message: string, status = 400, code?: string) {
  return NextResponse.json(
    { ok: false, error: { message, code } },
    { status },
  );
}
```

Clients then always branch on the same `{ ok }` discriminant.

------------------------------------------------------------------------
## STEP 2 : A Basic GET Handler

```ts
// app/api/health/route.ts
import { ok } from "@/lib/http";

// Opt out of caching for dynamic data.
export const dynamic = "force-dynamic";

export async function GET() {
  return ok({ status: "up", time: new Date().toISOString() });
}
```

Read query params from the request `URL`:

```ts
export async function GET(req: Request) {
  const { searchParams } = new URL(req.url);
  const q = searchParams.get("q") ?? "";
  return ok({ q });
}
```

------------------------------------------------------------------------
## STEP 3 : POST With Zod safeParse Validation

Never trust `await req.json()`. Parse it through a schema.

```ts
// app/api/users/route.ts
import { z } from "zod";
import { ok, err } from "@/lib/http";
import { auth } from "@/lib/auth";

const CreateUser = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(80),
});

export async function POST(req: Request) {
  // 1. authenticate
  const session = await auth();
  if (!session) return err("Unauthorized", 401);

  // 2. validate
  const body = await req.json().catch(() => null);
  const parsed = CreateUser.safeParse(body);
  if (!parsed.success) {
    return err("Invalid body", 422, "VALIDATION");
  }

  // 3. authorize
  if (!session.user.canCreate) return err("Forbidden", 403);

  // 4. do work
  const user = await db.user.create({ data: parsed.data });

  // 5. typed response
  return ok(user, { status: 201 });
}
```

`safeParse` never throws — you branch on `parsed.success` and keep full
control of the error shape.

------------------------------------------------------------------------
## STEP 4 : Dynamic Routes With Awaited Params

In Next.js 15+, `params` is a Promise — you MUST await it.

```ts
// app/api/users/[id]/route.ts
import { ok, err } from "@/lib/http";

export async function GET(
  _req: Request,
  { params }: { params: Promise<{ id: string }> },
) {
  const { id } = await params; // <- awaited
  const user = await db.user.findUnique({ where: { id } });
  if (!user) return err("Not found", 404);
  return ok(user);
}
```

```text
app/api/
 ├─ health/route.ts        →  GET /api/health
 ├─ users/route.ts         →  GET|POST /api/users
 └─ users/[id]/route.ts    →  GET /api/users/:id   (await params)
```

------------------------------------------------------------------------
## STEP 5 : Server Actions (mutations without a fetch)

For form mutations you often skip route handlers entirely and use a
Server Action. The `"use server"` directive marks a server-only function
callable directly from a client component.

```ts
// app/users/actions.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { auth } from "@/lib/auth";

const Schema = z.object({ name: z.string().min(1) });

export async function updateName(formData: FormData) {
  const session = await auth();                 // authenticate
  if (!session) throw new Error("Unauthorized");

  const parsed = Schema.safeParse({             // validate
    name: formData.get("name"),
  });
  if (!parsed.success) return { ok: false as const };

  await db.user.update({                        // do work
    where: { id: session.user.id },
    data: parsed.data,
  });

  revalidatePath("/users");                     // refresh cache
  return { ok: true as const };
}
```

Wire it to a form with zero client JS:

```tsx
import { updateName } from "./actions";

export default function Page() {
  return (
    <form action={updateName}>
      <input name="name" defaultValue="user@example.com" />
      <button type="submit">Save</button>
    </form>
  );
}
```

------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Follow the golden rule: authenticate → validate → authorize → work → respond
✓ Always safeParse request bodies — never trust req.json() directly
✓ await params in every dynamic [id] route (Next 15+ Promise params)
✓ Return one consistent shape: { ok, data } / { ok, error }
✓ Use ok()/err() helpers so status codes stay consistent
✓ Prefer Server Actions + revalidatePath for form mutations
✓ Set dynamic = "force-dynamic" when a GET must never be cached
✓ Keep Zod schemas colocated and export inferred types for clients
✓ Never leak internal error details — return safe messages only
```
