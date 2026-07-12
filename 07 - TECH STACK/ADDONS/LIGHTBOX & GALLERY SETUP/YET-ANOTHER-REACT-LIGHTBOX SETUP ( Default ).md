# YET-ANOTHER-REACT-LIGHTBOX SETUP [ LIGHTBOX & GALLERY ]
------------------------------------------------------------------------
Yet Another React Lightbox (yarl) is a modern, TypeScript-first lightbox
with a plugin architecture: zoom, thumbnails, fullscreen, slideshow,
captions, and video. It is the default because it is React-native, SSR
safe, accessible, and integrates cleanly with next/image.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add yet-another-react-lightbox
```
Windows note: none specific. Styles ship as a single CSS import; keep
the import path lowercase so case-sensitive CI builds match Windows dev.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  Gallery grid (thumbnails)                                |
|     click --> setIndex(i) + setOpen(true)                 |
|                                                           |
|  <Lightbox open index slides plugins=[Zoom, Thumbnails]>  |
|        |                                                  |
|        +-- Zoom        (pinch / scroll to zoom)           |
|        +-- Thumbnails  (filmstrip navigation)             |
|        +-- Fullscreen  (native fullscreen API)            |
|                                                           |
|  slides: [{ src, width, height, alt }]                    |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Gallery + Lightbox
```tsx
"use client";

import { useState } from "react";
import Lightbox from "yet-another-react-lightbox";
import "yet-another-react-lightbox/styles.css";

const slides = [
  { src: "/img/1.jpg", width: 1600, height: 900, alt: "One" },
  { src: "/img/2.jpg", width: 1600, height: 900, alt: "Two" },
];

export function Gallery() {
  const [index, setIndex] = useState(-1);

  return (
    <>
      <div className="grid grid-cols-3 gap-2">
        {slides.map((s, i) => (
          <button key={i} onClick={() => setIndex(i)}>
            <img src={s.src} alt={s.alt} className="h-40 w-full object-cover" />
          </button>
        ))}
      </div>

      <Lightbox
        open={index >= 0}
        index={index}
        close={() => setIndex(-1)}
        slides={slides}
      />
    </>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Enable Plugins
Import each plugin and its stylesheet, then pass it to `plugins`.
```tsx
import Zoom from "yet-another-react-lightbox/plugins/zoom";
import Thumbnails from "yet-another-react-lightbox/plugins/thumbnails";
import Fullscreen from "yet-another-react-lightbox/plugins/fullscreen";
import "yet-another-react-lightbox/plugins/thumbnails.css";

<Lightbox plugins={[Zoom, Thumbnails, Fullscreen]} /* ...props */ />;
```
------------------------------------------------------------------------
## STEP 5 : Integrate with next/image
Use a custom render slot so the lightbox serves optimized images.
```tsx
import Image from "next/image";

<Lightbox
  slides={slides}
  render={{
    slide: ({ slide }) => (
      <Image src={slide.src} alt={slide.alt ?? ""}
        width={slide.width} height={slide.height}
        style={{ objectFit: "contain" }} />
    ),
  }}
/>;
```
------------------------------------------------------------------------
## STEP 6 : Add Captions
```tsx
import Captions from "yet-another-react-lightbox/plugins/captions";
import "yet-another-react-lightbox/plugins/captions.css";

const captioned = slides.map((s) => ({ ...s, title: s.alt }));
<Lightbox plugins={[Captions]} slides={captioned} />;
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Import styles.css once; add each plugin's CSS alongside it
✓ Keep the lightbox in a "use client" component
✓ Provide width/height on every slide to prevent layout shift
✓ Use index = -1 as the closed sentinel for clean open/close
✓ Render slides via next/image for optimized delivery
✓ Always supply meaningful alt text for accessibility
✓ Load only the plugins you use to keep the bundle lean
✓ Lazy-load full-resolution images; show thumbnails in the grid
```
