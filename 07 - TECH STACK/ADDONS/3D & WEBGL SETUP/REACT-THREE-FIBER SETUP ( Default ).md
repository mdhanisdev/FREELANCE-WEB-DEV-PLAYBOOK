# REACT-THREE-FIBER SETUP [ 3D & WEBGL ]
------------------------------------------------------------------------

React Three Fiber (R3F) is a React renderer for Three.js: you describe a
WebGL scene declaratively with JSX and R3F reconciles it to Three objects.
Paired with drei helpers, it is the recommended default for 3D in React.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add three @react-three/fiber @react-three/drei
pnpm add -D @types/three
```

`three` is the engine, `@react-three/fiber` the renderer, `@react-three/drei`
a toolbox of ready-made cameras, controls, and loaders.

------------------------------------------------------------------------

## STEP 2 : Create the Canvas (Client)

WebGL runs only in the browser, so the scene must be a client component.

```tsx
// components/Scene.tsx
"use client";
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
import { SpinningBox } from "./SpinningBox";

export function Scene() {
  return (
    <Canvas
      camera={{ position: [3, 3, 3], fov: 50 }}
      className="h-[70vh] w-full rounded-lg bg-neutral-950"
    >
      <ambientLight intensity={0.4} />
      <directionalLight position={[5, 5, 5]} intensity={1} />
      <SpinningBox />
      <OrbitControls enableDamping />
    </Canvas>
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Animate with useFrame

```tsx
// components/SpinningBox.tsx
"use client";
import { useRef } from "react";
import { useFrame } from "@react-three/fiber";
import type { Mesh } from "three";

export function SpinningBox() {
  const ref = useRef<Mesh>(null);
  useFrame((_, delta) => {
    if (ref.current) ref.current.rotation.y += delta;
  });
  return (
    <mesh ref={ref}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#4f46e5" />
    </mesh>
  );
}
```

`useFrame` runs on every render tick; use `delta` for frame-rate
independent motion.

------------------------------------------------------------------------

## STEP 4 : Mount Without SSR

```tsx
// app/experience/page.tsx
import dynamic from "next/dynamic";

const Scene = dynamic(
  () => import("@/components/Scene").then((m) => m.Scene),
  { ssr: false }
);

export default function Experience() {
  return <main className="p-8"><Scene /></main>;
}
```

> Windows note: no native build step; R3F is pure JS. If a GPU driver
> issue blanks the canvas, update graphics drivers or test in Chrome.

------------------------------------------------------------------------

## STEP 5 : Render Graph

```text
  <Canvas>  (creates renderer + scene + camera + loop)
     |
     +-- <ambientLight/>      -> THREE.AmbientLight
     +-- <directionalLight/>  -> THREE.DirectionalLight
     +-- <mesh>               -> THREE.Mesh
     |     +-- <boxGeometry>  -> geometry
     |     +-- <meshStandardMaterial> -> material
     +-- <OrbitControls/>     -> drei helper
             |
   useFrame  ->  requestAnimationFrame loop mutates objects
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Render the Canvas client-side only (ssr:false via next/dynamic)
✓ Multiply motion by delta for frame-rate independence
✓ Reuse geometries/materials; avoid allocating inside useFrame
✓ Use drei helpers instead of hand-writing controls and loaders
✓ Dispose textures/models with useGLTF cache or on unmount
✓ Keep DOM state out of useFrame; mutate refs, not React state
✓ Set dpr={[1, 2]} on Canvas to cap pixel ratio on retina/4K
```
