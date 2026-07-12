# KEEN-SLIDER SETUP [ CAROUSEL & SLIDER ]
------------------------------------------------------------------------
Keen-Slider is a performant, dependency-free slider with a tiny
footprint and a clean hook API. It is fully headless like Embla but
leans into native pointer physics and a flexible plugin model. Good
default when you want smooth touch behaviour without extra CSS bundles.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add keen-slider
```
Windows note: import the single stylesheet `keen-slider/keen-slider.css`;
path casing matters on case-sensitive CI even though Windows tolerates
mismatches locally — keep it lowercase to avoid broken production builds.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  useKeenSlider(options, [plugins])                        |
|        |                                                  |
|        +--> sliderRef  --> attach to wrapper div          |
|        +--> instanceRef--> next / prev / moveToIdx        |
|                                                           |
|  <div ref={sliderRef} class="keen-slider">                |
|     <div class="keen-slider__slide"/>   slide 1           |
|     <div class="keen-slider__slide"/>   slide 2           |
|  </div>                                                   |
|                                                           |
|  plugins: autoplay, wheel-controls, resize hooks          |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Slider
```tsx
"use client";

import { useKeenSlider } from "keen-slider/react";
import "keen-slider/keen-slider.css";

export function Slider({ slides }: { slides: string[] }) {
  const [sliderRef] = useKeenSlider<HTMLDivElement>({
    loop: true,
    slides: { perView: 1, spacing: 16 },
  });

  return (
    <div ref={sliderRef} className="keen-slider">
      {slides.map((src, i) => (
        <div key={i} className="keen-slider__slide">
          <img src={src} alt="" className="h-64 w-full object-cover" />
        </div>
      ))}
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Responsive Slide Counts
```tsx
const [sliderRef] = useKeenSlider<HTMLDivElement>({
  slides: { perView: 1, spacing: 12 },
  breakpoints: {
    "(min-width: 768px)": { slides: { perView: 2, spacing: 16 } },
    "(min-width: 1024px)": { slides: { perView: 3, spacing: 24 } },
  },
});
```
------------------------------------------------------------------------
## STEP 5 : Add an Autoplay Plugin
Keen-Slider has no autoplay by default — you write a tiny plugin.
```tsx
"use client";

import type { KeenSliderInstance } from "keen-slider";

export function autoplay(delay = 4000) {
  return (slider: KeenSliderInstance) => {
    let timer: ReturnType<typeof setInterval>;
    const clear = () => clearInterval(timer);
    const run = () => {
      clear();
      timer = setInterval(() => slider.next(), delay);
    };
    slider.on("created", run);
    slider.on("dragStarted", clear);
    slider.on("animationEnded", run);
    slider.on("destroyed", clear);
  };
}
```
------------------------------------------------------------------------
## STEP 6 : Track the Active Slide
```tsx
const [current, setCurrent] = useState(0);
const [sliderRef] = useKeenSlider<HTMLDivElement>({
  slideChanged: (s) => setCurrent(s.track.details.rel),
});
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Import keen-slider/keen-slider.css exactly once, lowercase path
✓ Type the hook with <HTMLDivElement> for a correct ref
✓ Keep the slider inside a "use client" boundary
✓ Write small plugins for autoplay/wheel rather than pulling libs
✓ Always clear intervals on dragStarted and destroyed events
✓ Use the breakpoints option instead of manual media queries
✓ Read the active index from track.details.rel
✓ Call instanceRef.current?.update() after slides change
```
