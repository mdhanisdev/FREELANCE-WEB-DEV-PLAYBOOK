# PUPPETEER SETUP [ PDF ]
------------------------------------------------------------------------

Puppeteer drives a headless Chromium browser. For PDFs, you render an
HTML page (or a Next.js route) and call `page.pdf()`. Use it when you
want pixel-perfect output that matches real CSS — print stylesheets,
web fonts, complex layouts — rather than a component DSL.

------------------------------------------------------------------------

## STEP 1 : Install

Local/dev uses full Puppeteer; serverless uses a slim Chromium build:

```bash
pnpm add puppeteer
# serverless (Vercel/Lambda):
pnpm add puppeteer-core @sparticuz/chromium
```

------------------------------------------------------------------------

## STEP 2 : Rendering Pipeline

```text
+------------------------------------------------------------+
|  HTML string OR https://app/print-route                    |
|              |                                             |
|              v                                             |
|  chromium.launch -> page.setContent / page.goto            |
|              |                                             |
|              v                                             |
|  page.pdf({ format:"A4", printBackground:true })           |
|              |                                             |
|              v                                             |
|  Buffer -> Response(application/pdf)                        |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : A Reusable Generator

```ts
// lib/pdf.ts
import puppeteer from "puppeteer";

export async function htmlToPdf(html: string): Promise<Uint8Array> {
  const browser = await puppeteer.launch({
    headless: true,
    args: ["--no-sandbox", "--disable-setuid-sandbox"],
  });
  try {
    const page = await browser.newPage();
    await page.setContent(html, { waitUntil: "networkidle0" });
    return await page.pdf({
      format: "A4",
      printBackground: true,
      margin: { top: "20mm", bottom: "20mm", left: "15mm", right: "15mm" },
    });
  } finally {
    await browser.close();
  }
}
```

------------------------------------------------------------------------

## STEP 4 : Route Handler

```ts
// app/api/report/route.ts
import { htmlToPdf } from "@/lib/pdf";

export const runtime = "nodejs";
export const maxDuration = 30;

export async function GET() {
  const html = `<!doctype html><html><body style="font-family:sans-serif">
    <h1>Monthly Report</h1>
    <p>Generated ${new Date().toLocaleDateString("en-US")}</p>
  </body></html>`;

  const pdf = await htmlToPdf(html);
  return new Response(pdf, {
    headers: {
      "Content-Type": "application/pdf",
      "Content-Disposition": 'inline; filename="report.pdf"',
    },
  });
}
```

------------------------------------------------------------------------

## STEP 5 : Serverless Variant

On Vercel/Lambda, swap the launcher — bundled Chromium is too large:

```ts
import chromium from "@sparticuz/chromium";
import puppeteer from "puppeteer-core";

const browser = await puppeteer.launch({
  args: chromium.args,
  executablePath: await chromium.executablePath(),
  headless: true,
});
```

------------------------------------------------------------------------

## WINDOWS NOTES

`pnpm add puppeteer` downloads a Chromium build into the pnpm store on
first install — it can be slow behind a proxy. If Chromium fails to
launch on Windows, install with a system Chrome path and pass
`executablePath` to `launch`. The `--no-sandbox` args are safe locally.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always close the browser in a finally block to avoid leaks
✓ Set runtime="nodejs" and a maxDuration on the Route Handler
✓ Use waitUntil:"networkidle0" so fonts/images finish loading
✓ Enable printBackground:true or CSS backgrounds drop out
✓ Reuse one browser instance under load; don't launch per request
✓ Use puppeteer-core + @sparticuz/chromium on serverless
✓ Prefer @react-pdf/renderer when you don't need a real browser
✓ Sanitize any user HTML before setContent to prevent injection
```
