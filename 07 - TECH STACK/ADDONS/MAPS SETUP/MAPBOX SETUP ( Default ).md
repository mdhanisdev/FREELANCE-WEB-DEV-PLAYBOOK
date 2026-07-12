# MAPBOX SETUP [ MAPS ]
------------------------------------------------------------------------
Mapbox GL JS is our default mapping engine: vector tiles, GPU rendering,
custom styles, and a generous free tier. This guide wires it into a
Next.js App Router project with TypeScript and pnpm, keeping the token
server-scoped and the map component client-only.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add mapbox-gl react-map-gl
pnpm add -D @types/mapbox-gl
```

`react-map-gl` gives us a declarative React wrapper; `mapbox-gl` is the
underlying engine and ships its own CSS.
------------------------------------------------------------------------
## STEP 2 : Configure Environment

Create a public token in the Mapbox dashboard (URL-restricted in prod).

```bash
# .env.local
NEXT_PUBLIC_MAPBOX_TOKEN=pk.your_public_token_here
```

```text
Token scoping
  ┌──────────────────────────────────────────────┐
  │ pk.*  public  → browser, URL-restricted        │
  │ sk.*  secret  → server only, NEVER in NEXT_PUBLIC│
  └──────────────────────────────────────────────┘
```
------------------------------------------------------------------------
## STEP 3 : Import Global Styles

Add the engine stylesheet once in the root layout.

```tsx
// app/layout.tsx
import "mapbox-gl/dist/mapbox-gl.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Build the Map Component

The map touches `window`, so it must be a Client Component.

```tsx
// components/map.tsx
"use client";

import Map, { Marker, NavigationControl } from "react-map-gl";
import type { ViewState } from "react-map-gl";
import { useState } from "react";

const TOKEN = process.env.NEXT_PUBLIC_MAPBOX_TOKEN!;

export function MapView() {
  const [view, setView] = useState<Partial<ViewState>>({
    longitude: 2.3522,
    latitude: 48.8566,
    zoom: 11,
  });

  return (
    <Map
      {...view}
      onMove={(e) => setView(e.viewState)}
      mapboxAccessToken={TOKEN}
      mapStyle="mapbox://styles/mapbox/streets-v12"
      style={{ width: "100%", height: "100%" }}
    >
      <NavigationControl position="top-right" />
      <Marker longitude={2.3522} latitude={48.8566} color="#e11" />
    </Map>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Render Without SSR

The component must never render on the server.

```tsx
// app/map/page.tsx
import dynamic from "next/dynamic";

const MapView = dynamic(
  () => import("@/components/map").then((m) => m.MapView),
  { ssr: false, loading: () => <p>Loading map…</p> }
);

export default function Page() {
  return (
    <main style={{ height: "100dvh" }}>
      <MapView />
    </main>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# open http://localhost:3000/map
```

Windows note: `.env.local` line endings can end up CRLF; keep the token
on one line. If tiles 401, the token env var was not picked up — restart
`pnpm dev` after editing `.env.local` (Next only reads it at boot).
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use pk.* tokens in the browser, restricted by URL/domain
✓ Keep sk.* secret tokens server-side only, never NEXT_PUBLIC
✓ Render the map with ssr:false to avoid window errors
✓ Import mapbox-gl.css exactly once, in the root layout
✓ Debounce onMove writes if you persist view state
✓ Lazy-load heavy layers and cluster large point sets
✓ Pin the map to a fixed-height parent (100dvh, not 100vh)
✓ Rotate tokens on leak; monitor usage in the dashboard
```
