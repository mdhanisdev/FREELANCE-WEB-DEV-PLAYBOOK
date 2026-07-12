# REACT-PLAYER SETUP [ MEDIA PLAYER ]
------------------------------------------------------------------------

react-player is a single component that plays URLs from YouTube, Vimeo,
SoundCloud, HLS, DASH, and plain MP4 files. It is the fastest way to embed
mixed media sources without wiring up a player per provider.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add react-player
```

No API keys are needed - react-player wraps each provider's own embed or
the native `<video>` element.

------------------------------------------------------------------------

## STEP 2 : Client Component Wrapper

react-player touches the DOM, so it must be a client component. Import it
dynamically with SSR disabled to avoid hydration mismatches.

```tsx
// components/Player.tsx
"use client";
import dynamic from "next/dynamic";

const ReactPlayer = dynamic(() => import("react-player"), { ssr: false });

export function Player({ url }: { url: string }) {
  return (
    <div className="aspect-video w-full overflow-hidden rounded-lg">
      <ReactPlayer
        src={url}
        controls
        width="100%"
        height="100%"
      />
    </div>
  );
}
```

> Windows note: no native modules, so `pnpm install` works identically to
> macOS/Linux. Nothing to compile.

------------------------------------------------------------------------

## STEP 3 : Use It

```tsx
// app/watch/page.tsx
import { Player } from "@/components/Player";

export default function Watch() {
  return (
    <main className="mx-auto max-w-3xl p-8">
      <Player url="https://www.youtube.com/watch?v=dQw4w9WgXcQ" />
    </main>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Control and Events

```tsx
"use client";
import { useRef, useState } from "react";
import dynamic from "next/dynamic";
const ReactPlayer = dynamic(() => import("react-player"), { ssr: false });

export function ControlledPlayer({ url }: { url: string }) {
  const [playing, setPlaying] = useState(false);
  const ref = useRef<HTMLVideoElement>(null);
  return (
    <>
      <ReactPlayer
        ref={ref}
        src={url}
        playing={playing}
        onPlay={() => setPlaying(true)}
        onEnded={() => setPlaying(false)}
      />
      <button onClick={() => setPlaying((p) => !p)}>Toggle</button>
    </>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : How Source Routing Works

```text
   url prop
      |
      v
  +---------------------------------------------+
  |            react-player                     |
  |  detects provider from the URL pattern      |
  +---------------------------------------------+
     |          |          |            |
  YouTube    Vimeo      HLS/DASH     file (mp4)
  iframe     iframe     hls.js       <video>
```

Each source is lazy-loaded, so only the matched player code ships.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Load via next/dynamic with ssr:false to dodge hydration errors
✓ Wrap in an aspect-video container for responsive sizing
✓ Prefer controlled playing state over imperative play() calls
✓ Only enable light mode (thumbnail) for long lists to save data
✓ Respect autoplay policies: muted is required for autoplay
✓ Handle onError to fall back when a provider blocks embedding
✓ Keep the component client-only; never render it on the server
```
