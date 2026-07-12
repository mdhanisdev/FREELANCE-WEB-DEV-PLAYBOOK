# CSP MIDDLEWARE SETUP [ NONCE-BASED CONTENT SECURITY POLICY ]
------------------------------------------------------------------------
## STEP 1 : Why a Nonce-Based CSP

Content-Security-Policy is the single most effective defense against XSS.
A nonce lets you allow your own inline scripts while blocking injected
ones, so you never need `unsafe-inline`. Because the nonce is generated
per request, it must run in Edge Middleware.

```text
Request --> Middleware
              | generate random nonce (base64)
              | build CSP header referencing nonce
              v
        Set request header  x-nonce
        Set response header Content-Security-Policy
              |
              v
        Server Components read nonce, stamp <script nonce>
              |
              v
        Browser executes only nonce-matched scripts
```
------------------------------------------------------------------------
## STEP 2 : No Extra Dependencies

Middleware and `next/headers` are built into Next.js. Confirm your app
is on a version with stable middleware.

```bash
pnpm add next@latest
pnpm dev
```
------------------------------------------------------------------------
## STEP 3 : Create middleware.ts

Generate a nonce, assemble the policy, and forward it both ways.

```ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString("base64");

  const csp = [
    `default-src 'self'`,
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    `style-src 'self' 'nonce-${nonce}'`,
    `img-src 'self' data: https:`,
    `font-src 'self'`,
    `connect-src 'self'`,
    `frame-ancestors 'none'`,
    `base-uri 'self'`,
    `form-action 'self'`,
    `object-src 'none'`,
    `upgrade-insecure-requests`,
  ].join("; ");

  const requestHeaders = new Headers(request.headers);
  requestHeaders.set("x-nonce", nonce);

  const isProd = process.env.NODE_ENV === "production";
  const headerName = isProd
    ? "Content-Security-Policy-Report-Only"
    : "Content-Security-Policy";

  requestHeaders.set(headerName, csp);

  const response = NextResponse.next({ request: { headers: requestHeaders } });
  response.headers.set(headerName, csp);
  return response;
}

export const config = {
  matcher: [
    {
      source: "/((?!_next/static|_next/image|favicon.ico).*)",
      missing: [{ type: "header", key: "next-router-prefetch" }],
    },
  ],
};
```
------------------------------------------------------------------------
## STEP 4 : Consume the Nonce in the Root Layout

Read the nonce server-side and pass it to any inline script.

```tsx
import { headers } from "next/headers";
import Script from "next/script";

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const nonce = (await headers()).get("x-nonce") ?? "";
  return (
    <html lang="en">
      <body>
        {children}
        <Script
          id="analytics"
          nonce={nonce}
          strategy="afterInteractive"
          dangerouslySetInnerHTML={{ __html: `console.log("ready");` }}
        />
      </body>
    </html>
  );
}
```

Next.js automatically applies the `x-nonce` to its own framework scripts
when `strict-dynamic` is present, so hydration keeps working.
------------------------------------------------------------------------
## STEP 5 : Roll Out Report-Only First

Never enforce blind. Ship `Content-Security-Policy-Report-Only` to
production, collect violations, then flip to enforcing.

```text
PHASE 1  Report-Only in prod   -> observe console + report endpoint
PHASE 2  Fix every violation   -> remove inline handlers, add nonces
PHASE 3  Swap header name       -> Content-Security-Policy (enforce)
PHASE 4  Re-scan securityheaders.com -> confirm A+
```

Add a reporting endpoint to capture violations:

```ts
`report-uri /api/csp-report`,
`report-to csp-endpoint`,
```
------------------------------------------------------------------------
## STEP 6 : Avoid unsafe-inline

`unsafe-inline` silently defeats the whole policy — any injected inline
script would run. Use these substitutes instead.

```text
Inline onClick=""     -> attach listeners in a client component
Inline <style>        -> nonce it, or use CSS files / CSS modules
Third-party widgets   -> nonce their loader, trust via strict-dynamic
Tailwind runtime      -> compiled to a stylesheet, no inline needed
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Generate a fresh nonce per request inside middleware
✓ Use 'strict-dynamic' so trusted scripts can load their own deps
✓ Never include 'unsafe-inline' or 'unsafe-eval' in script-src
✓ Set frame-ancestors 'none' to replace X-Frame-Options
✓ Ship Report-Only in prod first, fix violations, then enforce
✓ Keep object-src 'none' and base-uri 'self' to block injection tricks
✓ Exclude static assets from the matcher to save Edge invocations
✓ Add upgrade-insecure-requests to catch stray http:// references
```
