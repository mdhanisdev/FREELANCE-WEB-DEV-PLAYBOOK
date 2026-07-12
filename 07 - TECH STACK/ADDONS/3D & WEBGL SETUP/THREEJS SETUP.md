# THREEJS SETUP [ 3D & WEBGL ]
------------------------------------------------------------------------

Three.js is the underlying WebGL engine. Using it directly (no React
renderer) gives maximum control over the render loop and is ideal when you
want an imperative scene managed by a single client component.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add three
pnpm add -D @types/three
```

------------------------------------------------------------------------

## STEP 2 : Imperative Scene in a Client Component

```tsx
// components/ThreeScene.tsx
"use client";
import { useEffect, useRef } from "react";
import * as THREE from "three";

export function ThreeScene() {
  const mountRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const mount = mountRef.current!;
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      50,
      mount.clientWidth / mount.clientHeight,
      0.1,
      100
    );
    camera.position.set(3, 3, 3);
    camera.lookAt(0, 0, 0);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(mount.clientWidth, mount.clientHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    mount.appendChild(renderer.domElement);

    const cube = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshStandardMaterial({ color: 0x4f46e5 })
    );
    scene.add(cube);
    scene.add(new THREE.AmbientLight(0xffffff, 0.4));
    const dir = new THREE.DirectionalLight(0xffffff, 1);
    dir.position.set(5, 5, 5);
    scene.add(dir);

    let raf = 0;
    const clock = new THREE.Clock();
    const animate = () => {
      cube.rotation.y += clock.getDelta();
      renderer.render(scene, camera);
      raf = requestAnimationFrame(animate);
    };
    animate();

    return () => {
      cancelAnimationFrame(raf);
      renderer.dispose();
      cube.geometry.dispose();
      (cube.material as THREE.Material).dispose();
      mount.removeChild(renderer.domElement);
    };
  }, []);

  return <div ref={mountRef} className="h-[70vh] w-full rounded-lg" />;
}
```

------------------------------------------------------------------------

## STEP 3 : Mount Without SSR

```tsx
// app/three/page.tsx
import dynamic from "next/dynamic";

const ThreeScene = dynamic(
  () => import("@/components/ThreeScene").then((m) => m.ThreeScene),
  { ssr: false }
);

export default function Page() {
  return <main className="p-8"><ThreeScene /></main>;
}
```

> Windows note: pure JS, no build tools. A blank canvas usually means a
> stale GPU driver - update it or test in a Chromium browser.

------------------------------------------------------------------------

## STEP 4 : The Core Loop

```text
  Scene  +  Camera  +  Renderer
     \       |         /
      \      |        /
       requestAnimationFrame
              |
       update objects (rotation, position)
              |
       renderer.render(scene, camera)
              |
        cleanup -> dispose() geometries,
        materials, renderer on unmount
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Run everything client-side; guard against SSR with ssr:false
✓ Always dispose geometries, materials, textures, and renderer
✓ Cancel the requestAnimationFrame on unmount to stop the loop
✓ Cap pixel ratio at 2 to protect performance on 4K/retina
✓ Use a single Clock delta for frame-rate independent motion
✓ Handle window resize: update camera aspect + renderer size
✓ Prefer BufferGeometry and reuse materials across meshes
```
