# SECURITY BEST PRACTICES [ NEXT.JS 16 ]

------------------------------------------------------------------------

## 1. Authentication

```text
✓ Use a trusted auth solution (e.g. Auth.js)
✓ Hash passwords (bcrypt/argon2)
✓ Never store plain-text passwords
✓ Use secure, httpOnly cookies for sessions
✓ Enable MFA where appropriate
```

------------------------------------------------------------------------

## 2. Authorization

```text
✓ Verify permissions on every protected API
✓ Implement Role-Based Access Control (RBAC)
✓ Never trust client-side role checks
```

------------------------------------------------------------------------

## 3. Input Validation

```text
✓ Validate all user input
✓ Use Zod for schema validation
✓ Validate query params, body, and route params
✓ Validate uploaded files
```

------------------------------------------------------------------------

## 4. Prevent XSS

```text
✓ Escape user input
✓ Avoid dangerouslySetInnerHTML
✓ Sanitize HTML if rendering user content
```

------------------------------------------------------------------------

## 5. Prevent CSRF

```text
✓ Use SameSite cookies
✓ Use CSRF tokens for state-changing requests
✓ Verify request origin when appropriate
```

------------------------------------------------------------------------

## 6. Prevent SQL Injection

```text
✓ Use Prisma/Drizzle ORM
✓ Use parameterized queries
✓ Never concatenate SQL strings
```

------------------------------------------------------------------------

## 7. Environment Variables

```text
✓ Store secrets in .env.local
✓ Never commit .env files
✓ Expose only NEXT_PUBLIC_* variables to the client
✓ Rotate secrets periodically
```

------------------------------------------------------------------------

## 8. Security Headers

```text
✓ Content-Security-Policy
✓ Strict-Transport-Security
✓ X-Content-Type-Options
✓ X-Frame-Options
✓ Referrer-Policy
```

------------------------------------------------------------------------

## 9. File Upload Security

```text
✓ Validate MIME type
✓ Limit file size
✓ Rename uploaded files
✓ Store uploads outside the public directory when possible
```

------------------------------------------------------------------------

## 10. Rate Limiting

```text
✓ Protect login endpoints
✓ Protect public APIs
✓ Apply middleware-based rate limiting
```

------------------------------------------------------------------------

## 11. Dependency Security

```bash
pnpm audit
pnpm update
```

```text
✓ Keep dependencies updated
✓ Remove unused packages
```

------------------------------------------------------------------------

## 12. Security Testing

```text
✓ OWASP ZAP
✓ pnpm audit
✓ Manual security checklist
✓ Playwright authentication flows
```

------------------------------------------------------------------------

## 13. Logging & Monitoring

```text
✓ Log authentication failures
✓ Monitor server errors
✓ Never log passwords, tokens or secrets
```

------------------------------------------------------------------------

## 14. Deployment Checklist

```text
□ HTTPS enabled
□ Environment variables configured
□ Security headers enabled
□ Production build tested
□ Dependencies audited
□ Sensitive routes protected
□ Backups configured
```

------------------------------------------------------------------------

## RECOMMENDED SECURITY WORKFLOW

```text
Write Secure Code
        │
        ▼
Validate Input (Zod)
        │
        ▼
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
