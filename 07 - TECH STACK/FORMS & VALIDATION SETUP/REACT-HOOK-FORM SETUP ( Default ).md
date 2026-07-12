# REACT-HOOK-FORM SETUP [ FORMS & VALIDATION ]
------------------------------------------------------------------------
## STEP 1 : Install
------------------------------------------------------------------------
Install react-hook-form together with Zod and the resolver bridge. The
resolver lets RHF consume a Zod schema for validation.

```bash
pnpm add react-hook-form zod @hookform/resolvers
```

------------------------------------------------------------------------
## STEP 2 : Define A Schema (Single Source Of Truth)
------------------------------------------------------------------------
Keep the schema in a shared file so the SAME rules run on the client
(instant UX feedback) and on the server (security). Never trust the
client alone.

```ts
// lib/schemas/sign-up.ts
import { z } from "zod";

export const signUpSchema = z
  .object({
    email: z.string().email("Enter a valid email"),
    password: z.string().min(8, "Min 8 characters"),
    confirm: z.string(),
  })
  .refine((d) => d.password === d.confirm, {
    message: "Passwords do not match",
    path: ["confirm"],
  });

export type SignUpValues = z.infer<typeof signUpSchema>;
```

------------------------------------------------------------------------
## STEP 3 : Wire useForm With zodResolver
------------------------------------------------------------------------
`register` binds native inputs, `handleSubmit` runs validation first, and
`formState` exposes errors plus the async `isSubmitting` flag.

```tsx
// components/sign-up-form.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { signUpSchema, type SignUpValues } from "@/lib/schemas/sign-up";

export function SignUpForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<SignUpValues>({
    resolver: zodResolver(signUpSchema),
    defaultValues: { email: "", password: "", confirm: "" },
  });

  async function onSubmit(values: SignUpValues) {
    // client is already valid here; server MUST re-check (Step 5)
    await fetch("/api/sign-up", {
      method: "POST",
      body: JSON.stringify(values),
    });
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <input placeholder="user@example.com" {...register("email")} />
      {errors.email && <p role="alert">{errors.email.message}</p>}

      <input type="password" {...register("password")} />
      {errors.password && <p role="alert">{errors.password.message}</p>}

      <input type="password" {...register("confirm")} />
      {errors.confirm && <p role="alert">{errors.confirm.message}</p>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Creating..." : "Create account"}
      </button>
    </form>
  );
}
```

------------------------------------------------------------------------
## STEP 4 : Validation & Submit Flow
------------------------------------------------------------------------
```text
  user types                 submit clicked
      |                            |
      v                            v
 +----------+   valid?  no   +-----------------+
 | register | ------------>  | errors rendered |
 +----------+                +-----------------+
      | yes                        ^
      v                            | invalid
 handleSubmit --> zodResolver -----+
      | valid
      v
 onSubmit() --> isSubmitting=true --> fetch --> SERVER re-validates
```

------------------------------------------------------------------------
## STEP 5 : Re-Validate On The Server (Security)
------------------------------------------------------------------------
Client validation is UX only. Anyone can POST raw JSON, so parse the same
schema again in the route handler before touching the database.

```ts
// app/api/sign-up/route.ts
import { NextResponse } from "next/server";
import { signUpSchema } from "@/lib/schemas/sign-up";

export async function POST(req: Request) {
  const parsed = signUpSchema.safeParse(await req.json());
  if (!parsed.success) {
    return NextResponse.json({ errors: parsed.error.flatten() }, { status: 400 });
  }
  // parsed.data is trusted and fully typed
  return NextResponse.json({ ok: true }, { status: 201 });
}
```

------------------------------------------------------------------------
## STEP 6 : shadcn/ui Form (Optional Layer)
------------------------------------------------------------------------
`pnpm dlx shadcn@latest add form` scaffolds `<Form>`, `<FormField>`,
`<FormMessage>` wrappers around RHF's `useForm`. It renders errors and
wires accessibility for you, so prefer it when using shadcn components.

------------------------------------------------------------------------
## BEST PRACTICES
------------------------------------------------------------------------
```text
✓ One Zod schema shared by client form AND server handler
✓ Always re-validate on the server; client checks are UX only
✓ Use defaultValues to keep inputs controlled from first render
✓ Disable the submit button with isSubmitting to stop double posts
✓ Add noValidate so RHF/Zod own the error UX, not the browser
✓ Render errors with role="alert" for screen readers
✓ Reach for shadcn <Form> when you already use shadcn/ui
✓ Never store or log real PII; use user@example.com in samples
```
