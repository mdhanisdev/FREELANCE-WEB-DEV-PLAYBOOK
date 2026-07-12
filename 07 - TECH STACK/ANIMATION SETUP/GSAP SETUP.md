# GSAP SETUP [ TIMELINE & SCROLL-DRIVEN ANIMATION ]
------------------------------------------------------------------------
## STEP 1 : When to Choose GSAP over Framer

Framer Motion is best for declarative component state (mount, exit,
gestures). Reach for GSAP when you need precise, imperative control:
long choreographed timelines, scrubbed scroll animations, SVG morphing,
and complex sequencing.

```text
Framer Motion            |  GSAP
-------------------------|------------------------------
component enter/exit     |  multi-step timelines
gesture state (hover)    |  scroll-scrubbed sequences
simple stagger           |  fine-grained overlap control
React-first, declarative |  imperative, framework-agnostic
```
------------------------------------------------------------------------
## STEP 2 : Install

```bash
pnpm add gsap @gsap/react
```

`@gsap/react` provides the `useGSAP` hook, which handles automatic
cleanup in React's lifecycle. All GSAP code runs client-side.
------------------------------------------------------------------------
## STEP 3 : Basic Tween with useGSAP

`useGSAP` scopes animations to a container ref and reverts them on
unmount — no manual teardown needed.

```tsx
"use client";
import { useRef } from "react";
import gsap from "gsap";
import { useGSAP } from "@gsap/react";

export function Hero() {
  const container = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      gsap.from(".headline", { y: 40, opacity: 0, duration: 0.6 });
    },
    { scope: container }
  );

  return (
    <div ref={container}>
      <h1 className="headline">Welcome</h1>
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Timelines for Sequencing

A timeline sequences tweens with relative or absolute positions. The
`"-=0.3"` overlaps the next tween into the previous by 0.3s.

```tsx
useGSAP(
  () => {
    const tl = gsap.timeline({ defaults: { ease: "power2.out" } });
    tl.from(".logo", { scale: 0, duration: 0.4 })
      .from(".title", { y: 30, opacity: 0 }, "-=0.2")
      .from(".cta", { opacity: 0 }, "-=0.1");
  },
  { scope: container }
);
```
------------------------------------------------------------------------
## STEP 5 : ScrollTrigger

Register the plugin once, then drive animations from scroll position.
Use `scrub: true` to tie progress to the scrollbar.

```tsx
"use client";
import { useRef } from "react";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { useGSAP } from "@gsap/react";

gsap.registerPlugin(ScrollTrigger, useGSAP);

export function Parallax() {
  const container = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      gsap.to(".panel", {
        yPercent: -50,
        ease: "none",
        scrollTrigger: {
          trigger: ".panel",
          start: "top bottom",
          end: "bottom top",
          scrub: true,
        },
      });
    },
    { scope: container }
  );

  return (
    <div ref={container}>
      <div className="panel">Parallax layer</div>
    </div>
  );
}
```

```text
scroll ->  start: "top bottom"        end: "bottom top"
             |                            |
  [====================== scrub 0..1 =====================]
             progress drives yPercel animation
```
------------------------------------------------------------------------
## STEP 6 : Cleanup with gsap.context

Outside `useGSAP` (e.g. plain useEffect), wrap animations in
`gsap.context` and revert in the cleanup function. This kills tweens and
ScrollTriggers created in the scope, preventing leaks on route changes.

```tsx
import { useEffect, useRef } from "react";
import gsap from "gsap";

useEffect(() => {
  const ctx = gsap.context(() => {
    gsap.to(".box", { rotation: 360, repeat: -1 });
  }, containerRef);

  return () => ctx.revert();
}, []);
```

`useGSAP` does this for you automatically; prefer it in React.
------------------------------------------------------------------------
## STEP 7 : Respect Reduced Motion

GSAP exposes `matchMedia` to gate animations behind media queries.

```tsx
useGSAP(() => {
  const mm = gsap.matchMedia();
  mm.add("(prefers-reduced-motion: no-preference)", () => {
    gsap.from(".headline", { y: 40, opacity: 0 });
  });
});
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Use the useGSAP hook for automatic React cleanup
✓ Always pass { scope: ref } so selectors stay contained
✓ registerPlugin(ScrollTrigger) once at module load
✓ Use scrub: true for scroll-linked, not scroll-triggered, motion
✓ Sequence with timelines instead of chained delays
✓ Revert with gsap.context outside useGSAP to avoid leaks
✓ Gate motion behind gsap.matchMedia for accessibility
✓ Choose GSAP for scrubbed/complex timelines, Framer for UI state
```
