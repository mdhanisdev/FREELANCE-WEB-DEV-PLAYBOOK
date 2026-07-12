# NEXTJS METADATA SETUP [ SEO, SOCIAL & STRUCTURED DATA ]
------------------------------------------------------------------------
## STEP 1 : The Metadata Surface

The App Router owns SEO through a typed Metadata API plus convention
files. There is nothing to install — it ships with Next.js.

```text
app/
 |- layout.tsx        -> static/root metadata (defaults)
 |- page.tsx          -> generateMetadata (dynamic per route)
 |- sitemap.ts        -> /sitemap.xml
 |- robots.ts         -> /robots.txt
 |- opengraph-image.tsx (or api route) -> dynamic OG image
```
------------------------------------------------------------------------
## STEP 2 : Root Metadata + metadataBase

Define global defaults once in the root layout. `metadataBase` makes all
relative OG/canonical URLs absolute.

```ts
import type { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL("https://example.com"),
  title: {
    default: "Acme — Ship Faster",
    template: "%s | Acme",
  },
  description: "Production-grade tooling for modern web teams.",
  openGraph: {
    type: "website",
    siteName: "Acme",
    locale: "en_US",
    images: ["/opengraph-image"],
  },
  twitter: {
    card: "summary_large_image",
    creator: "@acme",
  },
  alternates: { canonical: "/" },
};
```
------------------------------------------------------------------------
## STEP 3 : Dynamic generateMetadata

For content routes, resolve metadata from data. It runs on the server and
is deduped with the page's own data fetch.

```ts
import type { Metadata } from "next";

type Props = { params: Promise<{ slug: string }> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  return {
    title: post.title,
    description: post.excerpt,
    alternates: { canonical: `/blog/${slug}` },
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: "article",
      publishedTime: post.date,
      images: [`/blog/${slug}/opengraph-image`],
    },
  };
}
```
------------------------------------------------------------------------
## STEP 4 : sitemap.ts and robots.ts

Both are typed convention files that emit valid XML/text.

```ts
// app/sitemap.ts
import type { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts();
  const base = "https://example.com";

  return [
    { url: base, lastModified: new Date(), priority: 1 },
    ...posts.map((p) => ({
      url: `${base}/blog/${p.slug}`,
      lastModified: p.date,
      priority: 0.7,
    })),
  ];
}
```

```ts
// app/robots.ts
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: "*", allow: "/", disallow: "/admin" },
    sitemap: "https://example.com/sitemap.xml",
  };
}
```
------------------------------------------------------------------------
## STEP 5 : Dynamic OG Image with next/og

Generate share images at the edge with JSX. Place the file at
`app/opengraph-image.tsx` (or a `[slug]` variant).

```tsx
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          height: "100%",
          width: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          fontSize: 72,
          background: "#0a0a0a",
          color: "#fff",
        }}
      >
        Acme — Ship Faster
      </div>
    ),
    { ...size }
  );
}
```
------------------------------------------------------------------------
## STEP 6 : JSON-LD Structured Data

Inject schema.org JSON-LD in the page body. Serialize safely and let
search engines parse rich results.

```tsx
export default async function Page({ params }: Props) {
  const { slug } = await params;
  const post = await getPost(slug);

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    headline: post.title,
    datePublished: post.date,
    author: { "@type": "Organization", name: "Acme" },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>{/* ... */}</article>
    </>
  );
}
```
------------------------------------------------------------------------
## STEP 7 : Verify

```bash
pnpm build && pnpm start
```

Check `/sitemap.xml`, `/robots.txt`, view-source for `<meta>` and
JSON-LD, and validate rich results at search.google.com/test/rich-results.
Preview cards with the OpenGraph debuggers of each platform.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Set metadataBase once so relative OG/canonical URLs resolve absolutely
✓ Use a title template (%s | Brand) for consistent tab titles
✓ Prefer generateMetadata for any data-driven route
✓ Ship summary_large_image Twitter cards with 1200x630 OG images
✓ Always declare a canonical URL to avoid duplicate-content penalties
✓ Keep sitemap.ts data-driven; regenerate on publish
✓ Validate JSON-LD with the Rich Results Test before shipping
✓ Never inject unescaped user input into JSON-LD script tags
```
