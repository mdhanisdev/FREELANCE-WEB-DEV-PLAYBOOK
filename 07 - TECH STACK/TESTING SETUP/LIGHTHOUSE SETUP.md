# LIGHTHOUSE CLI SETUP [ PERFORMANCE, SEO & BEST PRACTICES ]

------------------------------------------------------------------------

## STEP 1 : Install Lighthouse

```bash
pnpm add -D lighthouse
```

------------------------------------------------------------------------

## STEP 2 : Create Reports Folder (Optional)

```text
reports/ś
```

------------------------------------------------------------------------

## STEP 3 : Update /package.json

```json
{
  "scripts": {
    "dev": "next dev",
    "lighthouse": "lighthouse http://localhost:3000 --output html --output json --output-path ./reports/lighthouse"
  }
}
```

------------------------------------------------------------------------

## STEP 4 : Start Your Next.js Application

```bash
pnpm dev
```

Make sure the application is running at:

```text
http://localhost:3000
```

------------------------------------------------------------------------

## STEP 5 : Run Lighthouse

Open another VS Code terminal and run:

```bash
pnpm lighthouse
```

------------------------------------------------------------------------

## STEP 6 : Generated Reports

```text
reports/
├── lighthouse.report.html
└── lighthouse.report.json
```

------------------------------------------------------------------------

## STEP 7 : Open the HTML Report

Open:

```text
reports/lighthouse.report.html
```

------------------------------------------------------------------------

## WHAT LIGHTHOUSE CHECKS

```text
Performance
│
├── First Contentful Paint (FCP)
├── Largest Contentful Paint (LCP)
├── Speed Index
├── Total Blocking Time (TBT)
└── Cumulative Layout Shift (CLS)

Accessibility
│
├── Color Contrast
├── ARIA Attributes
├── Labels
└── Keyboard Accessibility

Best Practices
│
├── HTTPS
├── Console Errors
├── Modern APIs
└── Security Checks

SEO
│
├── Meta Tags
├── Robots
├── Crawlability
└── Mobile Friendliness
```

------------------------------------------------------------------------

## RECOMMENDED WORKFLOW

```text
pnpm lint
      │
      ▼
pnpm test
      │
      ▼
pnpm e2e
      │
      ▼
pnpm lighthouse
      │
      ▼
Deploy
```
