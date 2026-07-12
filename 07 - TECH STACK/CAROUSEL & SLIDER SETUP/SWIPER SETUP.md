# SWIPER SETUP [ CAROUSEL & SLIDER ]
------------------------------------------------------------------------
Swiper is the most feature-complete touch slider: pagination, effects
(fade, cube, coverflow), thumbnails, virtual slides, and lazy loading
all built in. Choose it when you need rich, pre-styled behaviour fast
and are willing to ship a slightly larger bundle than Embla.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add swiper
```
Windows note: Swiper's CSS is imported from the package; if HMR fails to
pick up its styles on Windows, restart the dev server or enable polling
with `WATCHPACK_POLLING=true` in `.env.local`.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <Swiper modules=[Navigation, Pagination, Autoplay]>      |
|        |                                                  |
|        +-- <SwiperSlide/>   slide 1                       |
|        +-- <SwiperSlide/>   slide 2                       |
|        +-- <SwiperSlide/>   slide 3                       |
|                                                           |
|   Modules are opt-in --> tree-shaken from the bundle      |
|   onSwiper / onSlideChange --> imperative Swiper instance |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Slider
Swiper's React build is client-only in the App Router.
```tsx
"use client";

import { Swiper, SwiperSlide } from "swiper/react";
import { Navigation, Pagination, Autoplay } from "swiper/modules";
import "swiper/css";
import "swiper/css/navigation";
import "swiper/css/pagination";

export function Slider({ slides }: { slides: string[] }) {
  return (
    <Swiper
      modules={[Navigation, Pagination, Autoplay]}
      navigation
      pagination={{ clickable: true }}
      autoplay={{ delay: 4000, disableOnInteraction: true }}
      loop
      className="rounded-lg"
    >
      {slides.map((src, i) => (
        <SwiperSlide key={i}>
          <img src={src} alt="" className="h-64 w-full object-cover" />
        </SwiperSlide>
      ))}
    </Swiper>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Responsive Breakpoints
```tsx
<Swiper
  spaceBetween={16}
  slidesPerView={1}
  breakpoints={{
    768: { slidesPerView: 2 },
    1024: { slidesPerView: 3 },
  }}
>
  {/* slides */}
</Swiper>
```
------------------------------------------------------------------------
## STEP 5 : Access the Instance Imperatively
```tsx
"use client";

import { useRef } from "react";
import type { Swiper as SwiperClass } from "swiper";

export function Controlled() {
  const ref = useRef<SwiperClass | null>(null);
  return (
    <>
      <Swiper onSwiper={(s) => (ref.current = s)}>{/* slides */}</Swiper>
      <button onClick={() => ref.current?.slideNext()}>Next</button>
    </>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Effects and Lazy Loading
Import only the effect CSS you use to keep the bundle small.
```tsx
import { EffectFade } from "swiper/modules";
import "swiper/css/effect-fade";

<Swiper modules={[EffectFade]} effect="fade" />;
```
For images, add `loading="lazy"` on `<img>` inside each slide so
off-screen media does not block first paint.
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Import only the module CSS you actually use to trim the bundle
✓ Register features via the modules prop — nothing is global
✓ Keep <Swiper> in a "use client" component
✓ Prefer breakpoints prop over manual media queries
✓ Set disableOnInteraction so autoplay yields to the user
✓ Use loading="lazy" on slide images to protect LCP
✓ Reach for Embla instead when you need a smaller headless slider
✓ Store the instance via onSwiper for imperative control
```
