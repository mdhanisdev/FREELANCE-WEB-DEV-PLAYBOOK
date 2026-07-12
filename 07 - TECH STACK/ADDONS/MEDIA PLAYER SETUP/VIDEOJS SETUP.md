# VIDEOJS SETUP [ MEDIA PLAYER ]
------------------------------------------------------------------------

Video.js is a mature, plugin-rich HTML5 player with first-class HLS and
DASH support. You own the hosting; Video.js gives you a skinnable,
accessible UI and a large plugin ecosystem for ads, captions, and quality
selection.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add video.js
pnpm add -D @types/video.js
```

Video.js ships its own CSS, which you import once in your player module.

------------------------------------------------------------------------

## STEP 2 : Build a React Wrapper

Video.js manages its own DOM, so initialise it in an effect against a
ref and dispose it on unmount to avoid leaks.

```tsx
// components/VideoJS.tsx
"use client";
import { useEffect, useRef } from "react";
import videojs from "video.js";
import "video.js/dist/video-js.css";
import type Player from "video.js/dist/types/player";

export function VideoJS({ src }: { src: string }) {
  const ref = useRef<HTMLDivElement>(null);
  const playerRef = useRef<Player | null>(null);

  useEffect(() => {
    if (playerRef.current || !ref.current) return;
    const el = document.createElement("video-js");
    el.classList.add("vjs-big-play-centered");
    ref.current.appendChild(el);
    playerRef.current = videojs(el, {
      controls: true,
      responsive: true,
      fluid: true,
      sources: [{ src, type: "application/x-mpegURL" }],
    });
    return () => {
      playerRef.current?.dispose();
      playerRef.current = null;
    };
  }, [src]);

  return <div ref={ref} className="w-full rounded-lg overflow-hidden" />;
}
```

> Windows note: no native compilation; `pnpm install` behaves the same as
> on Unix. The `video-js.css` import path is case-sensitive on deploy.

------------------------------------------------------------------------

## STEP 3 : Use It

```tsx
// app/player/page.tsx
import { VideoJS } from "@/components/VideoJS";

export default function PlayerPage() {
  return (
    <main className="mx-auto max-w-3xl p-8">
      <VideoJS src="https://example.com/stream/master.m3u8" />
    </main>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Lifecycle Model

```text
  mount
    |
    v
  create <video-js> element -> videojs(el, options)
    |                              |
    |                          player instance (ref)
  interact / play / seek         |
    |                              v
  unmount -> player.dispose() -> tech + listeners freed
```

Never re-run `videojs()` on an already-initialised element; guard with the
ref as shown above.

------------------------------------------------------------------------

## STEP 5 : Add a Plugin

```bash
pnpm add @videojs/http-streaming
```

HLS/DASH support (VHS) is bundled by default in modern Video.js, so most
adaptive streams work out of the box. Add UI plugins (e.g. quality menu)
by importing them before the `videojs()` call.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Initialise once behind a ref guard; never re-init the same node
✓ Always call player.dispose() in the effect cleanup
✓ Import video-js.css exactly once at the player module
✓ Use fluid/responsive options instead of fixed pixel sizes
✓ Set correct MIME type (application/x-mpegURL for HLS)
✓ Keep the component client-only ("use client")
✓ Pin the video.js major version; test plugins against it
```
