# SECURITY TESTING SETUP [ OWASP ZAP ]

------------------------------------------------------------------------

## STEP 1 : Install OWASP ZAP

Download and install OWASP ZAP from the official website.

------------------------------------------------------------------------

## STEP 2 : Start Your Next.js App

```bash
pnpm dev
```

------------------------------------------------------------------------

## STEP 3 : Launch OWASP ZAP

Open OWASP ZAP and configure your browser (or use the built-in browser).

------------------------------------------------------------------------

## STEP 4 : Perform Automated Scan

1. Enter your application URL:

```text
http://localhost:3000
```

2. Run:

```text
Attack
└── Automated Scan
```

------------------------------------------------------------------------

## STEP 5 : Review Report

OWASP ZAP reports issues such as:

```text
✓ Missing Security Headers
✓ XSS
✓ CSRF
✓ Cookie Issues
✓ Information Disclosure
✓ Directory Listing
```

------------------------------------------------------------------------

## STEP 6 : Check Dependencies

```bash
pnpm audit
```

------------------------------------------------------------------------

## RECOMMENDED SECURITY WORKFLOW

```text
ESLint
   │
   ▼
pnpm audit
   │
   ▼
OWASP ZAP Scan
   │
   ▼
Fix Vulnerabilities
   │
   ▼
Deploy
```
