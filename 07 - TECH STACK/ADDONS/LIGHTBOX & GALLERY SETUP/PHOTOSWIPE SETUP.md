# PHOTOSWIPE SETUP [ LIGHTBOX & GALLERY ]
------------------------------------------------------------------------
PhotoSwipe is a battle-tested, framework-agnostic JavaScript image
gallery with best-in-class touch gestures and zoom performance. Choose
it for large, mobile-first photo galleries where native-feeling pinch
zoom and momentum matter more than deep React integration.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add photoswipe
```
Windows note: PhotoSwipe requires each anchor to declare real image
dimensions. Its CSS import path is lowercase — keep it exact so builds
that run on case-sensitive Linux CI match your Windows dev environment.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <a href data-pswp-width data-pswp-height>  gallery link  |
|  <a href data-pswp-width data-pswp-height>  gallery link  |
|        |                                                  |
|        v                                                  |
|  PhotoSwipeLightbox({ gallery, children, pswpModule })    |
|        |                                                  |
|        +-- lightbox.init()   in useEffect                 |
|        +-- lightbox.destroy() in cleanup                  |
|                                                           |
|  click link --> full-screen PhotoSwipe viewer             |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Gallery Component
PhotoSwipe is imperative, so wire it up in an effect on the client.
```tsx
"use client";

import { useEffect, useRef } from "react";
import PhotoSwipeLightbox from "photoswipe/lightbox";
import "photoswipe/style.css";

type Photo = { src: string; w: number; h: number; alt: string };

export function Gallery({ photos }: { photos: Photo[] }) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const lightbox = new PhotoSwipeLightbox({
      gallery: ref.current!,
      children: "a",
      pswpModule: () => import("photoswipe"),
    });
    lightbox.init();
    return () => lightbox.destroy();
  }, []);

  return (
    <div ref={ref} className="grid grid-cols-3 gap-2">
      {photos.map((p, i) => (
        <a key={i} href={p.src} data-pswp-width={p.w} data-pswp-height={p.h}
          target="_blank" rel="noreferrer">
          <img src={p.src} alt={p.alt} className="h-40 w-full object-cover" />
        </a>
      ))}
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Add Captions Dynamically
```tsx
lightbox.on("uiRegister", () => {
  lightbox.pswp?.ui?.registerElement({
    name: "caption",
    order: 9,
    isButton: false,
    appendTo: "root",
    onInit: (el, pswp) => {
      pswp.on("change", () => {
        const cur = pswp.currSlide?.data.element as HTMLElement;
        el.textContent = cur?.querySelector("img")?.alt ?? "";
      });
    },
  });
});
```
------------------------------------------------------------------------
## STEP 5 : Tune Zoom Behaviour
```tsx
const lightbox = new PhotoSwipeLightbox({
  gallery: ref.current!,
  children: "a",
  initialZoomLevel: "fit",
  secondaryZoomLevel: 2,
  maxZoomLevel: 4,
  pswpModule: () => import("photoswipe"),
});
```
------------------------------------------------------------------------
## STEP 6 : Dimensions Are Mandatory
PhotoSwipe needs `data-pswp-width` / `data-pswp-height` on every anchor.
If images are user-uploaded, capture dimensions at upload time and store
them so the gallery never has to measure them in the browser.
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Always init in useEffect and destroy() in the cleanup return
✓ Provide data-pswp-width/height on every gallery anchor
✓ Import "photoswipe" lazily via pswpModule for code-splitting
✓ Keep the gallery in a "use client" component
✓ Import style.css once, at the gallery module level
✓ Store image dimensions server-side for user-uploaded media
✓ Give every <img> descriptive alt text for accessibility
✓ Prefer yet-another-react-lightbox for deeper React integration
```
