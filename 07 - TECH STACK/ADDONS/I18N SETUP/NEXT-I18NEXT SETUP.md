# NEXT-I18NEXT SETUP [ I18N ]
------------------------------------------------------------------------
i18next is the alternative when you want the mature i18next ecosystem:
pluralization, ICU, backends, and a huge plugin catalog. This guide uses
`react-i18next` with an App Router-friendly server init in TypeScript and
pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add i18next react-i18next i18next-resources-to-backend
```
------------------------------------------------------------------------
## STEP 2 : Declare Supported Locales

```ts
// i18n/settings.ts
export const languages = ["en", "fr", "de"] as const;
export const defaultLng = "en";
export const defaultNS = "common";

export function options(lng: string, ns = defaultNS) {
  return { lng, ns, fallbackLng: defaultLng, supportedLngs: languages };
}
```

```text
Resource layout
  locales/
    en/common.json
    fr/common.json
    de/common.json
```
------------------------------------------------------------------------
## STEP 3 : Create a Server Init Factory

```ts
// i18n/server.ts
import { createInstance } from "i18next";
import resourcesToBackend from "i18next-resources-to-backend";
import { initReactI18next } from "react-i18next/initReactI18next";
import { options } from "./settings";

export async function initI18n(lng: string, ns?: string) {
  const i18n = createInstance();
  await i18n
    .use(initReactI18next)
    .use(
      resourcesToBackend(
        (l: string, n: string) => import(`../locales/${l}/${n}.json`)
      )
    )
    .init(options(lng, ns));
  return i18n;
}
```
------------------------------------------------------------------------
## STEP 4 : Translate in a Server Component

```tsx
// app/[locale]/page.tsx
import { initI18n } from "@/i18n/server";

export default async function Home({
  params,
}: {
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  const i18n = await initI18n(locale);
  const t = i18n.getFixedT(locale, "common");
  return <h1>{t("title")}</h1>;
}
```

```json
// locales/en/common.json
{ "title": "Welcome", "cta": "Get started" }
```
------------------------------------------------------------------------
## STEP 5 : Add a Client Provider

```tsx
// components/i18n-provider.tsx
"use client";

import { I18nextProvider } from "react-i18next";
import { createInstance } from "i18next";
import { initReactI18next } from "react-i18next/initReactI18next";
import { options } from "@/i18n/settings";

export function I18nProvider({
  lng,
  resources,
  children,
}: {
  lng: string;
  resources: Record<string, unknown>;
  children: React.ReactNode;
}) {
  const i18n = createInstance();
  i18n.use(initReactI18next).init({ ...options(lng), resources });
  return <I18nextProvider i18n={i18n}>{children}</I18nextProvider>;
}
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# open http://localhost:3000/en
```

Windows note: the dynamic `import(`../locales/${l}/${n}.json`)` path uses
forward slashes and resolves fine on Windows. Keep folder names in the
exact locale casing your links use.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Create a fresh i18next instance per request on the server
✓ Never share a mutable global instance across requests
✓ Split translations into namespaces (common, auth, errors)
✓ Always configure fallbackLng and supportedLngs
✓ Use getFixedT on the server to avoid hook constraints
✓ Lazy-load namespaces via resources-to-backend
✓ Keep locale folder names lowercase and consistent
✓ Lean on i18next plurals/ICU instead of manual string logic
```
