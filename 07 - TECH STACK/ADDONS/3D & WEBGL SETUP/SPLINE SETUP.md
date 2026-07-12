# SPLINE SETUP [ 3D & WEBGL ]
------------------------------------------------------------------------

Spline is a browser-based 3D design tool. You model and animate a scene in
the Spline editor, export it, and embed it in React with a single
runtime component - no Three.js code required. Best for designer-driven 3D
where you want visuals without hand-writing WebGL.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add @splinetool/react-spline @splinetool/runtime
```

`@splinetool/react-spline` is the React wrapper; `@splinetool/runtime` is
the WebGL engine that draws the exported scene.

------------------------------------------------------------------------

## STEP 2 : Export a Scene

In the Spline editor choose Export -> Code (React) or Export -> Public URL.
You get a `.splinecode` URL like:

```text
https://prod.spline.design/AbCdEf123456/scene.splinecode
```

------------------------------------------------------------------------

## STEP 3 : Embed the Scene (Client)

The runtime needs the browser, so import it dynamically without SSR.

```tsx
// components/SplineScene.tsx
"use client";
import dynamic from "next/dynamic";

const Spline = dynamic(() => import("@splinetool/react-spline"), {
  ssr: false,
});

export function SplineScene() {
  return (
    <div className="h-[80vh] w-full">
      <Spline scene="https://prod.spline.design/AbCdEf123456/scene.splinecode" />
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : React to Scene Events

Spline objects can emit events you handle in React, and you can call
methods on named objects.

```tsx
"use client";
import dynamic from "next/dynamic";
import type { Application } from "@splinetool/runtime";
const Spline = dynamic(() => import("@splinetool/react-spline"), {
  ssr: false,
});

export function Interactive() {
  function onLoad(app: Application) {
    const obj = app.findObjectByName("Button");
    if (obj) obj.position.x = 1;
  }
  return <Spline scene="/scene.splinecode" onLoad={onLoad} />;
}
```

> Windows note: nothing compiles locally; the heavy lifting is the WebGL
> runtime in the browser. Large scenes load slowly on low-end GPUs.

------------------------------------------------------------------------

## STEP 5 : How It Renders

```text
  Spline editor  --export-->  scene.splinecode (assets + logic)
                                     |
                                     v
   <Spline scene=...> mounts @splinetool/runtime
                                     |
                          WebGL canvas draws the scene
                                     |
        onLoad(app) -> findObjectByName -> mutate/animate
```

------------------------------------------------------------------------

## STEP 6 : Performance

Add a loading placeholder and lazy-mount below the fold, since scenes can
be several megabytes.

```tsx
<Spline scene="/scene.splinecode" className="opacity-0 transition
  duration-500 [&.loaded]:opacity-100" />
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Import the runtime with ssr:false; it is browser-only
✓ Show a skeleton/placeholder while the .splinecode loads
✓ Lazy-mount heavy scenes below the fold to protect LCP
✓ Keep scene polygon/texture budgets low for mobile GPUs
✓ Use onLoad + findObjectByName for interaction, not DOM hacks
✓ Host .splinecode on a CDN and cache it aggressively
✓ Provide a static image fallback for reduced-motion users
```
