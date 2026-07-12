# SECURITY HEADERS SETUP [ HARDEN HTTP RESPONSES ]
------------------------------------------------------------------------
## STEP 1 : Why HTTP Security Headers

Security headers instruct the browser how to treat your site. They are a
cheap, high-impact defense layer that mitigates clickjacking, MIME
sniffing, protocol downgrade, and referrer leakage. They live in
`next.config.ts` so every route inherits them.

```text
   Client Request
        |
        v
+---------------------+       +--------------------------+
|  Next.js Server     | ----> |  Response + Headers      |
|  (next.config.ts)   |       |  HSTS / X-Frame / ...    |
+---------------------+       +--------------------------+
                                        |
                                        v
                               Browser enforces policy
```
------------------------------------------------------------------------
## STEP 2 : Baseline (No New Dependencies)

Security headers ship with Next.js core, so no install is required. If
you are starting fresh:

```bash
pnpm create next-app@latest my-app --ts --app
cd my-app
pnpm install
```
------------------------------------------------------------------------
## STEP 3 : Define Headers in next.config.ts

Add an async `headers()` function that returns a rule for all paths.

```ts
import type { NextConfig } from "next";

const securityHeaders = [
  {
    key: "Strict-Transport-Security",
    value: "max-age=63072000; includeSubDomains; preload",
  },
  { key: "X-Frame-Options", value: "SAMEORIGIN" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  {
    key: "Permissions-Policy",
    value: "camera=(), microphone=(), geolocation=(), browsing-topics=()",
  },
];

const nextConfig: NextConfig = {
  async headers() {
    return [{ source: "/:path*", headers: securityHeaders }];
  },
};

export default nextConfig;
```
------------------------------------------------------------------------
## STEP 4 : What Each Header Does

```text
Strict-Transport-Security  -> Force HTTPS, block downgrade attacks
X-Frame-Options            -> Stop your pages being framed (clickjack)
X-Content-Type-Options     -> Disable MIME sniffing
Referrer-Policy            -> Trim referrer sent to other origins
Permissions-Policy         -> Deny access to sensitive browser APIs
```

HSTS `preload` is a commitment: only enable once HTTPS is guaranteed on
the apex domain and all subdomains, then submit to hstspreload.org.
------------------------------------------------------------------------
## STEP 5 : Run and Inspect Locally

```bash
pnpm dev
```

Inspect the emitted headers with curl (Git Bash on Windows):

```bash
curl -sI http://localhost:3000 | grep -Ei "strict|x-frame|x-content|referrer|permissions"
```

You should see all five headers echoed back on every route.
------------------------------------------------------------------------
## STEP 6 : Verify with securityheaders.com

After deploying to a public HTTPS URL:

1. Open https://securityheaders.com
2. Enter your production domain and scan.
3. Aim for grade A. Missing CSP is the usual reason you do not get A+.
4. Re-scan after every header change; results are cached ~a few minutes.

```text
+------------------------------------------------+
|  securityheaders.com   Grade:  A  ->  A+       |
|  [x] Strict-Transport-Security                 |
|  [x] X-Frame-Options                           |
|  [x] X-Content-Type-Options                    |
|  [x] Referrer-Policy                           |
|  [x] Permissions-Policy                        |
|  [ ] Content-Security-Policy  (see CSP setup)  |
+------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 7 : Per-Route Overrides

You can scope stricter headers to a subtree by adding more entries with a
narrower `source`. Later, more specific rules do not override earlier
ones automatically, so keep values complete per matcher.

```ts
return [
  { source: "/:path*", headers: securityHeaders },
  {
    source: "/admin/:path*",
    headers: [{ key: "X-Frame-Options", value: "DENY" }],
  },
];
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Apply headers to /:path* so no route is ever missed
✓ Ship HSTS with includeSubDomains; only add preload when 100% HTTPS
✓ Keep Permissions-Policy deny-by-default, opt in features explicitly
✓ Use strict-origin-when-cross-origin for Referrer-Policy
✓ Verify on securityheaders.com after each deploy, target A+
✓ Pair these headers with a nonce-based CSP (separate setup) for A+
✓ Never rely on X-Frame-Options alone; add frame-ancestors in CSP
✓ Re-test with curl in CI to catch accidental header regressions
```
