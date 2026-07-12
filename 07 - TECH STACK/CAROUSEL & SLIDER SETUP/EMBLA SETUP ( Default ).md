# EMBLA SETUP [ CAROUSEL & SLIDER ]
------------------------------------------------------------------------
Embla Carousel is a lightweight, dependency-free carousel with a
precise React hook API. It is headless — you own the markup and styles —
which makes it the default choice for a Tailwind stack where design
consistency matters. Great physics, tiny bundle, excellent a11y hooks.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add embla-carousel-react
pnpm add embla-carousel-autoplay
```
Windows note: none specific — Embla is pure JS. Standard pnpm install
behaviour applies across drives.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  useEmblaCarousel(options, [plugins])                     |
|        |                                                  |
|        +--> emblaRef  --> attach to VIEWPORT div          |
|        +--> emblaApi  --> scrollNext / scrollTo / on()    |
|                                                           |
|  <div ref={emblaRef} class="overflow-hidden">   viewport  |
|     <div class="flex">                           container |
|        <div class="min-w-0 flex-[0_0_100%]"/>    slide    |
|        <div class="min-w-0 flex-[0_0_100%]"/>    slide    |
|     </div>                                                |
|  </div>                                                   |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Carousel
```tsx
"use client";

import useEmblaCarousel from "embla-carousel-react";
import Autoplay from "embla-carousel-autoplay";

export function Carousel({ slides }: { slides: string[] }) {
  const [emblaRef] = useEmblaCarousel(
    { loop: true, align: "start" },
    [Autoplay({ delay: 4000, stopOnInteraction: true })]
  );

  return (
    <div className="overflow-hidden" ref={emblaRef}>
      <div className="flex">
        {slides.map((src, i) => (
          <div key={i} className="min-w-0 flex-[0_0_100%] p-2">
            <img src={src} alt="" className="h-64 w-full object-cover" />
          </div>
        ))}
      </div>
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Add Prev / Next Controls
```tsx
"use client";

import { useCallback } from "react";
import useEmblaCarousel from "embla-carousel-react";

export function WithButtons() {
  const [emblaRef, emblaApi] = useEmblaCarousel({ loop: true });
  const prev = useCallback(() => emblaApi?.scrollPrev(), [emblaApi]);
  const next = useCallback(() => emblaApi?.scrollNext(), [emblaApi]);

  return (
    <div className="relative">
      <div className="overflow-hidden" ref={emblaRef}>
        <div className="flex">{/* slides */}</div>
      </div>
      <button onClick={prev} className="absolute left-2 top-1/2">Prev</button>
      <button onClick={next} className="absolute right-2 top-1/2">Next</button>
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Sync Dot Indicators
```tsx
"use client";

import { useEffect, useState } from "react";

export function useDots(emblaApi: any) {
  const [selected, setSelected] = useState(0);
  useEffect(() => {
    if (!emblaApi) return;
    const onSelect = () => setSelected(emblaApi.selectedScrollSnap());
    emblaApi.on("select", onSelect);
    onSelect();
    return () => emblaApi.off("select", onSelect);
  }, [emblaApi]);
  return selected;
}
```
------------------------------------------------------------------------
## STEP 6 : Responsive Slide Widths
Change `flex-basis` per breakpoint with Tailwind to show multiple slides.
```tsx
<div className="min-w-0 flex-[0_0_100%] md:flex-[0_0_50%] lg:flex-[0_0_33%]" />
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Mark the carousel "use client" — hooks need the browser
✓ Keep the viewport (overflow-hidden) and container (flex) separate
✓ Always clean up emblaApi.on() listeners in the effect return
✓ Set stopOnInteraction on Autoplay so users keep control
✓ Add aria-label + keyboard handlers for accessible navigation
✓ Use flex-basis, not fixed widths, for responsive slide counts
✓ reInit the API after async slide data loads
✓ Lazy-load off-screen slide images to protect LCP
```
