# NEXT-INTL SETUP [ I18N ]
------------------------------------------------------------------------
next-intl is our default internationalization layer: built for the App
Router, it supports Server Components, locale-prefixed routing, and typed
message keys. This guide covers a `[locale]` segment setup with
TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add next-intl
```
------------------------------------------------------------------------
## STEP 2 : Define Routing

```ts
// i18n/routing.ts
import { defineRouting } from "next-intl/routing";

export const routing = defineRouting({
  locales: ["en", "fr", "de"],
  defaultLocale: "en",
});
```

```text
URL layout
  /            → redirect → /en
  /en/about    → English
  /fr/about    → French
  /de/about    → German
```
------------------------------------------------------------------------
## STEP 3 : Add Middleware

```ts
// middleware.ts
import createMiddleware from "next-intl/middleware";
import { routing } from "@/i18n/routing";

export default createMiddleware(routing);

export const config = {
  matcher: ["/((?!api|_next|_vercel|.*\\..*).*)"],
};
```
------------------------------------------------------------------------
## STEP 4 : Wire the Request Config

```ts
// i18n/request.ts
import { getRequestConfig } from "next-intl/server";
import { routing } from "@/i18n/routing";

export default getRequestConfig(async ({ requestLocale }) => {
  let locale = await requestLocale;
  if (!locale || !routing.locales.includes(locale as any)) {
    locale = routing.defaultLocale;
  }
  return {
    locale,
    messages: (await import(`../messages/${locale}.json`)).default,
  };
});
```

```json
// messages/en.json
{
  "Home": {
    "title": "Welcome",
    "cta": "Get started"
  }
}
```
------------------------------------------------------------------------
## STEP 5 : Add the Provider Layout

```tsx
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from "next-intl";
import { getMessages } from "next-intl/server";
import { notFound } from "next/navigation";
import { routing } from "@/i18n/routing";

export default async function LocaleLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  if (!routing.locales.includes(locale as any)) notFound();
  const messages = await getMessages();

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Use Translations

```tsx
// app/[locale]/page.tsx  (Server Component)
import { useTranslations } from "next-intl";

export default function Home() {
  const t = useTranslations("Home");
  return <h1>{t("title")}</h1>;
}
```

Enable the plugin in `next.config.ts` with `createNextIntlPlugin()`.
Windows note: dynamic `import(`../messages/${locale}.json`)` resolves the
same on Windows; keep filenames lowercase to match locale codes exactly.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Keep all routes under the [locale] segment
✓ Exclude api/_next/static assets in the middleware matcher
✓ Validate the incoming locale, notFound() on mismatch
✓ Use useTranslations in Server Components where possible
✓ Namespace messages by feature (Home, Auth, Errors)
✓ Enable typed messages for compile-time key safety
✓ Store one JSON file per locale, lowercase filenames
✓ Set <html lang> from the active locale for a11y + SEO
```
