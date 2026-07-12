# SERWIST SETUP [ PWA & OFFLINE ]

------------------------------------------------------------------------

Serwist is the maintained successor to next-pwa / Workbox. It compiles a
service worker, wires precaching + runtime caching, and turns a Next.js
App Router app into an installable, offline-capable PWA.

------------------------------------------------------------------------

## STEP 1 : Install Packages

```bash
pnpm add @serwist/next
pnpm add -D serwist
```

------------------------------------------------------------------------

## STEP 2 : Wrap next.config

```ts
// next.config.ts
import withSerwistInit from "@serwist/next";

const withSerwist = withSerwistInit({
  swSrc: "app/sw.ts",
  swDest: "public/sw.js",
  disable: process.env.NODE_ENV === "development",
});

export default withSerwist({});
```

Disabling in development avoids caching your hot-reload assets.

------------------------------------------------------------------------

## STEP 3 : Author The Service Worker

```ts
// app/sw.ts
import { defaultCache } from "@serwist/next/worker";
import { Serwist } from "serwist";

declare const self: ServiceWorkerGlobalScope & {
  __SW_MANIFEST: (string | { url: string; revision: string })[];
};

const serwist = new Serwist({
  precacheEntries: self.__SW_MANIFEST,
  skipWaiting: true,
  clientsClaim: true,
  navigationPreload: true,
  runtimeCaching: defaultCache,
  fallbacks: {
    entries: [{ url: "/offline", matcher: ({ request }) => request.mode === "navigate" }],
  },
});

serwist.addEventListeners();
```

------------------------------------------------------------------------

## STEP 4 : Add A Web App Manifest

Next.js generates the manifest from a metadata route.

```ts
// app/manifest.ts
import type { MetadataRoute } from "next";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "Example App",
    short_name: "Example",
    start_url: "/",
    display: "standalone",
    background_color: "#ffffff",
    theme_color: "#000000",
    icons: [
      { src: "/icon-192.png", sizes: "192x192", type: "image/png" },
      { src: "/icon-512.png", sizes: "512x512", type: "image/png" },
    ],
  };
}
```

Add a simple `app/offline/page.tsx` to serve as the navigation fallback.

------------------------------------------------------------------------

## STEP 5 : Caching Architecture

```text
  First visit
  -----------
  build -> __SW_MANIFEST (hashed assets)
  install sw -> precache shell + static assets

  Repeat / offline
  ----------------
  request -> sw fetch handler
     precached ? -> serve from cache
     runtime rule match ? -> stale-while-revalidate
     navigation + offline ? -> /offline fallback
     else -> network
```

------------------------------------------------------------------------

## STEP 6 : Windows Notes

Keep `disable: true` in development on Windows so the service worker does
not cache dev bundles and mask edits during `pnpm dev`. Test installs
against a production build: `pnpm build && pnpm start`. Service workers
require HTTPS in production; `localhost` is exempt on Windows browsers.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Disable the SW in development to avoid stale cached bundles
✓ Provide an /offline fallback route for navigations
✓ Ship 192px and 512px icons plus a maskable variant
✓ Use skipWaiting + clientsClaim so updates activate promptly
✓ Choose runtime cache strategies per resource type deliberately
✓ Test installability and offline in a production build, not dev
✓ Bump asset revisions so users receive fresh content on deploy
```
