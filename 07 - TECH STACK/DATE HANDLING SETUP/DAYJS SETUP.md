# DAYJS SETUP [ DATE HANDLING ]
------------------------------------------------------------------------

Day.js is a 2 KB immutable date library with a Moment.js-compatible
API. It ships a minimal core and adds features through plugins, so you
opt into exactly the functionality you need. Great when you want a
chainable, familiar API without Moment's bulk.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add dayjs
```

The core is tiny; relative time, timezone, and custom parsing come from
plugins you extend on.

------------------------------------------------------------------------

## STEP 2 : Plugin Model

```text
+------------------------------------------------------------+
|  dayjs (core: parse / format / add / diff)                 |
|      + extend(relativeTime)  -> .fromNow()                 |
|      + extend(utc)           -> .utc()                     |
|      + extend(timezone)      -> .tz("America/New_York")    |
|      + extend(customParse)   -> dayjs(str, "DD/MM/YYYY")   |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Configure Once

```ts
// lib/dayjs.ts
import dayjs from "dayjs";
import utc from "dayjs/plugin/utc";
import timezone from "dayjs/plugin/timezone";
import relativeTime from "dayjs/plugin/relativeTime";

dayjs.extend(utc);
dayjs.extend(timezone);
dayjs.extend(relativeTime);

export default dayjs;
```

Import this configured instance everywhere instead of the raw package
so plugins are guaranteed to be registered.

------------------------------------------------------------------------

## STEP 4 : Helpers

```ts
// lib/date.ts
import dayjs from "@/lib/dayjs";

export const iso = (d: string | Date) => dayjs(d).format("YYYY-MM-DD");

export const relative = (d: string | Date) => dayjs(d).fromNow();

export const inZone = (d: string | Date, tz: string) =>
  dayjs(d).tz(tz).format("YYYY-MM-DD HH:mm z");

export const addDays = (d: string | Date, n: number) =>
  dayjs(d).add(n, "day").toDate();
```

------------------------------------------------------------------------

## STEP 5 : Use in a Component

```tsx
// components/timestamp.tsx
import { relative } from "@/lib/date";

export function Timestamp({ value }: { value: string }) {
  return (
    <time dateTime={value} className="text-muted-foreground">
      {relative(value)}
    </time>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Locale-Aware Display

Day.js has its own locales, but for currency-adjacent UI you can still
lean on `Intl.DateTimeFormat` with `.toDate()`:

```ts
new Intl.DateTimeFormat("en-US", { dateStyle: "long" }).format(
  dayjs("2026-07-12").toDate()
);
```

------------------------------------------------------------------------

## WINDOWS NOTES

No native modules. The `timezone` plugin relies on the runtime's Intl
time-zone data; Node on Windows bundles full ICU, so zone conversions
match your Linux deploy target. Keep source timestamps in UTC.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Extend plugins once in lib/dayjs.ts, import that instance
✓ Treat Day.js objects as immutable — every op returns a new one
✓ Store UTC; convert to the user's zone only for display
✓ Load only the plugins you use to keep the bundle tiny
✓ Use fromNow() for relative labels, format() for absolute
✓ Prefer explicit format tokens over relying on locale defaults
✓ Wrap parsing of non-ISO strings with the customParseFormat plugin
✓ Pick Day.js when migrating from Moment for API familiarity
```
