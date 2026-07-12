# ZOD SETUP [ SCHEMA VALIDATION & TYPE INFERENCE ]
------------------------------------------------------------------------
## STEP 1 : Install
------------------------------------------------------------------------
Zod is a TypeScript-first schema library. One schema gives you runtime
validation AND a static type via inference.

```bash
pnpm add zod
```

------------------------------------------------------------------------
## STEP 2 : Define A Schema + Infer The Type
------------------------------------------------------------------------
Write the schema once, then derive the TypeScript type with `z.infer`.
The type and the validator can never drift apart.

```ts
// lib/schemas/user.ts
import { z } from "zod";

export const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(80),
  age: z.number().int().min(13).optional(),
  role: z.enum(["admin", "member"]).default("member"),
});

export type User = z.infer<typeof userSchema>;
```

------------------------------------------------------------------------
## STEP 3 : parse vs safeParse
------------------------------------------------------------------------
`parse` throws on invalid input; `safeParse` returns a discriminated
result you can branch on without try/catch. Prefer `safeParse` at trust
boundaries (requests, env, external APIs).

```ts
const result = userSchema.safeParse(input);
if (!result.success) {
  console.error(result.error.flatten());
} else {
  result.data; // typed as User, guaranteed valid
}
```

------------------------------------------------------------------------
## STEP 4 : Reuse One Schema — Client + Server
------------------------------------------------------------------------
```text
                lib/schemas/user.ts
                        |
        +---------------+----------------+
        |                                |
   CLIENT (form)                    SERVER (route/action)
   zodResolver(schema)              schema.safeParse(payload)
   instant UX errors                security gate before DB
        |                                |
        +--------> same rules <----------+
```

Client-side (react-hook-form) for fast feedback:

```tsx
"use client";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { userSchema, type User } from "@/lib/schemas/user";

const form = useForm<User>({ resolver: zodResolver(userSchema) });
```

Server-side (Route Handler) — the real security check:

```ts
// app/api/users/route.ts
import { NextResponse } from "next/server";
import { userSchema } from "@/lib/schemas/user";

export async function POST(req: Request) {
  const parsed = userSchema.safeParse(await req.json());
  if (!parsed.success) {
    return NextResponse.json({ errors: parsed.error.flatten() }, { status: 400 });
  }
  return NextResponse.json({ user: parsed.data }, { status: 201 });
}
```

Server Action variant — validate the FormData before mutating:

```ts
"use server";
import { userSchema } from "@/lib/schemas/user";

export async function createUser(formData: FormData) {
  const parsed = userSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { ok: false, errors: parsed.error.flatten() };
  // trusted parsed.data
  return { ok: true };
}
```

------------------------------------------------------------------------
## STEP 5 : Common Validators & Transforms
------------------------------------------------------------------------
```ts
z.string().email();                     // user@example.com
z.string().min(8).max(100);             // length bounds
z.string().url();                       // valid URL
z.string().regex(/^\+?[0-9]{7,15}$/);   // phone-ish
z.coerce.number().positive();           // "42" -> 42
z.coerce.date();                        // "2026-07-12" -> Date
z.boolean().default(false);
z.array(z.string()).nonempty();
z.enum(["light", "dark"]);
z.object({ a: z.string() }).partial();  // all keys optional
schema.refine((v) => cond, { message, path }); // cross-field
```

Validate environment variables once at startup:

```ts
// lib/env.ts
import { z } from "zod";
export const env = z
  .object({ DATABASE_URL: z.string().url(), NODE_ENV: z.string() })
  .parse(process.env);
```

------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Keep schemas in lib/schemas and import everywhere
✓ Derive types with z.infer — never hand-write the interface
✓ Use safeParse at every trust boundary (request, env, API)
✓ Validate on the client for UX AND re-validate on the server
✓ Use z.coerce for query params and FormData string inputs
✓ Attach .refine() with a path for cross-field rules
✓ flatten() the error for clean field-level messages
✓ Parse process.env once so a bad deploy fails fast
✓ Never place real PII in examples; use user@example.com
```
