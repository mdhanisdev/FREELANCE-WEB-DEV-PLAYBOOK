# GOOGLE-MAPS SETUP [ MAPS ]
------------------------------------------------------------------------
Google Maps is the alternative when you need Street View, rich POI data,
or the Places/Directions APIs. This guide uses the official
`@vis.gl/react-google-maps` wrapper inside Next.js App Router with
TypeScript and pnpm.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies

```bash
pnpm add @vis.gl/react-google-maps
```

The wrapper handles async script loading, so no manual `<script>` tag is
required.
------------------------------------------------------------------------
## STEP 2 : Configure Environment

Create an API key in Google Cloud, enable "Maps JavaScript API", and
restrict it by HTTP referrer.

```bash
# .env.local
NEXT_PUBLIC_GOOGLE_MAPS_KEY=AIza_your_key_here
NEXT_PUBLIC_GOOGLE_MAPS_ID=your_map_id
```

```text
Google Cloud checklist
  ┌────────────────────────────────────────────┐
  │ 1. Enable Maps JavaScript API                │
  │ 2. Create API key                            │
  │ 3. Restrict → HTTP referrers (your domains)  │
  │ 4. Create a Map ID for cloud-based styling   │
  └────────────────────────────────────────────┘
```
------------------------------------------------------------------------
## STEP 3 : Build the Map Component

```tsx
// components/google-map.tsx
"use client";

import {
  APIProvider,
  Map,
  AdvancedMarker,
} from "@vis.gl/react-google-maps";

const KEY = process.env.NEXT_PUBLIC_GOOGLE_MAPS_KEY!;
const MAP_ID = process.env.NEXT_PUBLIC_GOOGLE_MAPS_ID!;

export function GoogleMapView() {
  return (
    <APIProvider apiKey={KEY}>
      <Map
        mapId={MAP_ID}
        defaultCenter={{ lat: 48.8566, lng: 2.3522 }}
        defaultZoom={11}
        gestureHandling="greedy"
        disableDefaultUI={false}
        style={{ width: "100%", height: "100%" }}
      >
        <AdvancedMarker position={{ lat: 48.8566, lng: 2.3522 }} />
      </Map>
    </APIProvider>
  );
}
```

`AdvancedMarker` requires a `mapId`; without one it silently falls back.
------------------------------------------------------------------------
## STEP 4 : Render the Page

The wrapper is client-only, but `APIProvider` guards `window`, so a
plain client import works.

```tsx
// app/map/page.tsx
import { GoogleMapView } from "@/components/google-map";

export default function Page() {
  return (
    <main style={{ height: "100dvh" }}>
      <GoogleMapView />
    </main>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Add Places Autocomplete (Optional)

Server-side geocoding keeps your key off the wire.

```ts
// app/api/geocode/route.ts
import { NextResponse } from "next/server";

export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get("q");
  const key = process.env.GOOGLE_MAPS_SERVER_KEY!; // separate, IP-restricted
  const res = await fetch(
    `https://maps.googleapis.com/maps/api/geocode/json?address=${q}&key=${key}`
  );
  return NextResponse.json(await res.json());
}
```
------------------------------------------------------------------------
## STEP 6 : Verify Locally

```bash
pnpm dev
# open http://localhost:3000/map
```

Windows note: PowerShell may block pasting long keys with quotes — edit
`.env.local` in your editor instead. If the map is grey, the referrer
restriction is blocking `localhost`; add `http://localhost:3000/*`.
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Restrict the browser key by HTTP referrer, always
✓ Use a separate IP-restricted key for server geocoding
✓ Enable only the APIs you actually call (least privilege)
✓ Provide a Map ID so AdvancedMarker renders correctly
✓ Set a billing budget + alert to avoid surprise charges
✓ Cache geocode/directions results to cut quota usage
✓ Prefer cloud-based map styling over inline style arrays
✓ Pin the map to a fixed-height parent element
```
