# LEAFLET SETUP [ MAPS ]
------------------------------------------------------------------------
Leaflet is the lightweight, open-source choice: no API key, raster tiles
from OpenStreetMap, and a tiny footprint. This guide wires
`react-leaflet` into Next.js App Router with TypeScript and pnpm, working
around its hard dependency on the browser DOM.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add leaflet react-leaflet
pnpm add -D @types/leaflet
```
------------------------------------------------------------------------
## STEP 2 : Import Global Styles

```tsx
// app/layout.tsx
import "leaflet/dist/leaflet.css";

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

The tile grid collapses without this stylesheet.
------------------------------------------------------------------------
## STEP 3 : Fix the Default Marker Icons

Leaflet resolves icon URLs relative to the bundle and breaks under
webpack. Patch it once.

```ts
// lib/leaflet-icon.ts
import L from "leaflet";
import marker from "leaflet/dist/images/marker-icon.png";
import marker2x from "leaflet/dist/images/marker-icon-2x.png";
import shadow from "leaflet/dist/images/marker-shadow.png";

L.Icon.Default.mergeOptions({
  iconRetinaUrl: marker2x.src,
  iconUrl: marker.src,
  shadowUrl: shadow.src,
});
```
------------------------------------------------------------------------
## STEP 4 : Build the Map Component

```tsx
// components/leaflet-map.tsx
"use client";

import "@/lib/leaflet-icon";
import { MapContainer, TileLayer, Marker, Popup } from "react-leaflet";

export function LeafletMap() {
  return (
    <MapContainer
      center={[48.8566, 2.3522]}
      zoom={11}
      style={{ width: "100%", height: "100%" }}
    >
      <TileLayer
        attribution='&copy; OpenStreetMap contributors'
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      />
      <Marker position={[48.8566, 2.3522]}>
        <Popup>City center</Popup>
      </Marker>
    </MapContainer>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Render Without SSR

Leaflet reads `window` at import time, so dynamic import with
`ssr:false` is mandatory.

```tsx
// app/map/page.tsx
import dynamic from "next/dynamic";

const LeafletMap = dynamic(
  () => import("@/components/leaflet-map").then((m) => m.LeafletMap),
  { ssr: false, loading: () => <p>Loading map…</p> }
);

export default function Page() {
  return (
    <main style={{ height: "100dvh" }}>
      <LeafletMap />
    </main>
  );
}
```

```text
Render flow
  page (server) ──dynamic(ssr:false)──▶ LeafletMap (client)
                                          └─ window ok ✓
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# open http://localhost:3000/map
```

Windows note: file-watching for the PNG icon imports can lag on some
setups; a `pnpm dev` restart clears a stale icon 404. If markers show a
broken image, the Step 3 patch was not imported before the map.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Always render Leaflet with ssr:false (window at import time)
✓ Patch default marker icons once in a shared module
✓ Keep OpenStreetMap attribution visible (license requirement)
✓ Use a paid/self-hosted tile provider for heavy production traffic
✓ Constrain zoom + bounds to avoid pointless tile fetches
✓ Import leaflet.css exactly once, in the root layout
✓ Cluster large marker sets with leaflet.markercluster
✓ Pin MapContainer to a fixed-height parent
```
