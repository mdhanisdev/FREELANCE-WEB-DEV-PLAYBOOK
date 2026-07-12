# VALIBOT SETUP [ LIGHTWEIGHT SCHEMA VALIDATION ]
------------------------------------------------------------------------
## STEP 1 : Install
------------------------------------------------------------------------
Valibot is a modular, tree-shakeable validation library. Its API is
function-based instead of Zod's chained methods, so bundlers drop every
validator you do not import.

```bash
pnpm add valibot
pnpm add @hookform/resolvers react-hook-form
```

------------------------------------------------------------------------
## STEP 2 : Why Choose Valibot Over Zod
------------------------------------------------------------------------
```text
              ZOD                         VALIBOT
  +---------------------------+  +---------------------------+
  | chained: z.string().email()|  | piped: pipe(string(),email())
  | one large module          |  | many small functions      |
  | ~12 kb+ in bundle         |  | ~1-2 kb (tree-shaken)     |
  | huge ecosystem            |  | growing ecosystem         |
  +---------------------------+  +---------------------------+

  Pick Valibot when client bundle size matters (edge, mobile,
  many forms). Pick Zod when you want the largest ecosystem and
  do not ship the schema to the client.
```

------------------------------------------------------------------------
## STEP 3 : Define A Schema + Infer The Type
------------------------------------------------------------------------
Validators are composed with `pipe`. Derive the type with `InferOutput`.

```ts
// lib/schemas/sign-up.ts
import * as v from "valibot";

export const signUpSchema = v.object({
  email: v.pipe(v.string(), v.email("Enter a valid email")),
  password: v.pipe(v.string(), v.minLength(8, "Min 8 characters")),
  age: v.optional(v.pipe(v.number(), v.integer(), v.minValue(13))),
});

export type SignUpValues = v.InferOutput<typeof signUpSchema>;
```

------------------------------------------------------------------------
## STEP 4 : parse vs safeParse
------------------------------------------------------------------------
`parse` throws on invalid input; `safeParse` returns an object with a
`success` flag plus `output` or `issues`. Use `safeParse` at trust
boundaries.

```ts
import * as v from "valibot";
import { signUpSchema } from "@/lib/schemas/sign-up";

const result = v.safeParse(signUpSchema, input);
if (!result.success) {
  console.error(v.flatten(result.issues)); // field-level messages
} else {
  result.output; // typed + validated
}
```

------------------------------------------------------------------------
## STEP 5 : Client Form (UX) With react-hook-form
------------------------------------------------------------------------
```tsx
"use client";
import { useForm } from "react-hook-form";
import { valibotResolver } from "@hookform/resolvers/valibot";
import { signUpSchema, type SignUpValues } from "@/lib/schemas/sign-up";

export function SignUpForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<SignUpValues>({ resolver: valibotResolver(signUpSchema) });

  return (
    <form onSubmit={handleSubmit(async (v) => { /* POST to server */ })}>
      <input placeholder="user@example.com" {...register("email")} />
      {errors.email && <p role="alert">{errors.email.message}</p>}
      <input type="password" {...register("password")} />
      {errors.password && <p role="alert">{errors.password.message}</p>}
      <button disabled={isSubmitting}>Sign up</button>
    </form>
  );
}
```

------------------------------------------------------------------------
## STEP 6 : Server Re-Validation (Security)
------------------------------------------------------------------------
The client check is UX only. Re-run the SAME schema in the route handler
before trusting the payload — a raw POST bypasses the browser entirely.

```ts
// app/api/sign-up/route.ts
import { NextResponse } from "next/server";
import * as v from "valibot";
import { signUpSchema } from "@/lib/schemas/sign-up";

export async function POST(req: Request) {
  const result = v.safeParse(signUpSchema, await req.json());
  if (!result.success) {
    return NextResponse.json({ errors: v.flatten(result.issues) }, { status: 400 });
  }
  return NextResponse.json({ ok: true }, { status: 201 });
}
```

------------------------------------------------------------------------
## STEP 7 : Common Validators
------------------------------------------------------------------------
```ts
v.pipe(v.string(), v.email());
v.pipe(v.string(), v.minLength(8), v.maxLength(100));
v.pipe(v.string(), v.url());
v.pipe(v.string(), v.regex(/^\+?[0-9]{7,15}$/));
v.pipe(v.number(), v.integer(), v.minValue(0));
v.picklist(["light", "dark"]);
v.optional(v.string());
v.array(v.string());
```

------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ Choose Valibot when client bundle size is the priority
✓ Import only the validators you use so tree-shaking works
✓ Share one schema for the form AND the server handler
✓ Validate on the client for UX AND re-validate on the server
✓ Use safeParse at trust boundaries; parse only when a throw is fine
✓ Derive types with v.InferOutput — never hand-write them
✓ flatten(issues) for clean per-field error messages
✓ Keep secrets and real PII out of examples; use user@example.com
```
