# FRAMER MOTION SETUP [ DECLARATIVE UI ANIMATION ]
------------------------------------------------------------------------
## STEP 1 : Install the motion Package

Framer Motion for React now ships as the `motion` package. Import from
`motion/react`.

```bash
pnpm add motion
```

Animated components must be client components — they use hooks and DOM
refs.

```text
"use client"  <-- required at top of any file using motion/react
```
------------------------------------------------------------------------
## STEP 2 : Fade In on Mount

The core primitive is `motion.<tag>` with `initial`, `animate`, and
`transition`.

```tsx
"use client";
import { motion } from "motion/react";

export function FadeIn({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 16 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.4, ease: "easeOut" }}
    >
      {children}
    </motion.div>
  );
}
```
------------------------------------------------------------------------
## STEP 3 : Scroll-Triggered with whileInView

Animate when an element scrolls into the viewport. `once: true` prevents
replays; `amount` sets how much must be visible.

```tsx
<motion.section
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, amount: 0.3 }}
  transition={{ duration: 0.5 }}
>
  Reveals when 30% enters the viewport
</motion.section>
```

```text
   viewport
 +-----------------+
 |                 |
 |   [ card ] <----|-- 30% visible -> animation fires once
 |                 |
 +-----------------+
```
------------------------------------------------------------------------
## STEP 4 : Staggered Children

Use variants and `staggerChildren` on a parent to cascade children.

```tsx
"use client";
import { motion } from "motion/react";

const container = {
  hidden: {},
  show: { transition: { staggerChildren: 0.08 } },
};
const item = {
  hidden: { opacity: 0, y: 12 },
  show: { opacity: 1, y: 0 },
};

export function List({ items }: { items: string[] }) {
  return (
    <motion.ul variants={container} initial="hidden" animate="show">
      {items.map((t) => (
        <motion.li key={t} variants={item}>
          {t}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Exit Animations with AnimatePresence

Wrap conditionally rendered nodes so they animate out before unmounting.
Every direct child needs a stable `key`.

```tsx
"use client";
import { AnimatePresence, motion } from "motion/react";

export function Toast({ open }: { open: boolean }) {
  return (
    <AnimatePresence>
      {open && (
        <motion.div
          key="toast"
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: 20 }}
        >
          Saved!
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Hover and Tap Gestures

Declarative gesture props run on the compositor when possible.

```tsx
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 25 }}
>
  Click me
</motion.button>
```
------------------------------------------------------------------------
## STEP 7 : Respect Reduced Motion

Honor OS-level "reduce motion" so animations never harm accessibility.

```tsx
"use client";
import { motion, useReducedMotion } from "motion/react";

export function Reveal({ children }: { children: React.ReactNode }) {
  const reduce = useReducedMotion();
  return (
    <motion.div
      initial={reduce ? false : { opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
    >
      {children}
    </motion.div>
  );
}
```
------------------------------------------------------------------------
## BEST PRACTICES

```text
✓ Mark every motion file "use client"
✓ Animate transform + opacity only; they are GPU-accelerated
✓ Use variants + staggerChildren instead of manual delays
✓ Give AnimatePresence children a stable, unique key
✓ Prefer viewport={{ once: true }} to avoid replay jank
✓ Always branch on useReducedMotion for accessibility
✓ Use spring for gestures, tween/duration for entrances
✓ Keep transition durations 0.2s–0.5s for UI; longer feels sluggish
```
