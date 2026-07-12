# REACT-PDF SETUP [ PDF ]
------------------------------------------------------------------------

`@react-pdf/renderer` lets you build PDFs declaratively with React
components (`Document`, `Page`, `View`, `Text`). It renders to a real
PDF byte stream on the server — perfect for invoices, receipts, and
reports generated inside a Next.js Route Handler.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @react-pdf/renderer
```

This is the renderer package (not the `react-pdf` viewer). It runs in
Node, so keep it out of Client Components.

------------------------------------------------------------------------

## STEP 2 : Flow

```text
+------------------------------------------------------------+
|  Route Handler (Node runtime)                              |
|      |                                                     |
|      v                                                     |
|  <Document>  ->  renderToStream() / renderToBuffer()       |
|      |                                                     |
|      v                                                     |
|  application/pdf  ->  Response  ->  browser download       |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Define the Document

```tsx
// components/pdf/invoice-doc.tsx
import {
  Document,
  Page,
  View,
  Text,
  StyleSheet,
} from "@react-pdf/renderer";

const styles = StyleSheet.create({
  page: { padding: 40, fontSize: 12 },
  header: { fontSize: 20, marginBottom: 16 },
  row: { flexDirection: "row", justifyContent: "space-between" },
});

export function InvoiceDoc({
  customer,
  amount,
}: {
  customer: string;
  amount: number;
}) {
  return (
    <Document>
      <Page size="A4" style={styles.page}>
        <Text style={styles.header}>Invoice</Text>
        <View style={styles.row}>
          <Text>Customer</Text>
          <Text>{customer}</Text>
        </View>
        <View style={styles.row}>
          <Text>Amount</Text>
          <Text>
            {new Intl.NumberFormat("en-US", {
              style: "currency",
              currency: "USD",
            }).format(amount)}
          </Text>
        </View>
      </Page>
    </Document>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Stream It From a Route Handler

```ts
// app/api/invoice/route.ts
import { renderToStream } from "@react-pdf/renderer";
import { InvoiceDoc } from "@/components/pdf/invoice-doc";

export const runtime = "nodejs";

export async function GET() {
  const stream = await renderToStream(
    <InvoiceDoc customer="Acme Corp" amount={1250} />
  );

  return new Response(stream as unknown as ReadableStream, {
    headers: {
      "Content-Type": "application/pdf",
      "Content-Disposition": 'attachment; filename="invoice.pdf"',
    },
  });
}
```

`export const runtime = "nodejs"` is required — the renderer relies on
Node streams and will not run on the Edge runtime.

------------------------------------------------------------------------

## STEP 5 : Trigger a Download

```tsx
// components/pdf/download-button.tsx
"use client";
export function DownloadButton() {
  return (
    <a
      href="/api/invoice"
      className="rounded bg-blue-600 px-4 py-2 text-white"
    >
      Download PDF
    </a>
  );
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

No headless browser needed, so nothing to install natively on Windows.
Custom fonts must be registered with `Font.register` and loaded from a
URL or an absolute path — use `path.join(process.cwd(), ...)` rather
than hard-coded backslashes.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep the renderer server-only — never import into a client bundle
✓ Set runtime = "nodejs" on the Route Handler
✓ Prefer renderToStream for large docs, renderToBuffer for small
✓ Build layouts with View + flexDirection, not CSS grid
✓ Register fonts once at module load to avoid repeated fetches
✓ Format currency/dates with Intl before passing into <Text>
✓ Set Content-Disposition to control download vs inline view
✓ Pass plain data props so documents stay easy to snapshot-test
```
