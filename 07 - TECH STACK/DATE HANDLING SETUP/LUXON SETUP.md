# LUXON SETUP [ DATE HANDLING ]
------------------------------------------------------------------------

Luxon is a modern date/time library built on the native `Intl` API. Its
`DateTime` type is immutable and time-zone aware by design, making it
the strongest choice when your app is heavily zone- and locale-driven
(scheduling, calendars, multi-region reporting).

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add luxon
pnpm add -D @types/luxon
```

Luxon uses the platform Intl data for zones and locales, so there is no
separate tz database to bundle.

------------------------------------------------------------------------

## STEP 2 : Core Concepts

```text
+------------------------------------------------------------+
|  DateTime  (immutable, zone-aware)                         |
|     .fromISO("2026-07-12T18:00Z", { zone })                |
|     .setZone("America/New_York")                           |
|     .plus({ days: 3 })                                     |
|     .toFormat("yyyy-LL-dd HH:mm")                          |
|                                                            |
|  Duration  (spans)     Interval  (start -> end ranges)     |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Helpers Module

```ts
// lib/date.ts
import { DateTime } from "luxon";

export const iso = (d: string) =>
  DateTime.fromISO(d).toFormat("yyyy-LL-dd");

export const inZone = (d: string, zone: string) =>
  DateTime.fromISO(d, { zone: "utc" })
    .setZone(zone)
    .toFormat("yyyy-LL-dd HH:mm ZZZZ");

export const relative = (d: string) =>
  DateTime.fromISO(d).toRelative(); // "in 3 days"

export const addDays = (d: string, n: number) =>
  DateTime.fromISO(d).plus({ days: n }).toJSDate();
```

------------------------------------------------------------------------

## STEP 4 : Locale-Aware Formatting

Luxon wraps `Intl`, so localized output is a first-class feature:

```ts
import { DateTime } from "luxon";

DateTime.fromISO("2026-07-12")
  .setLocale("en-US")
  .toLocaleString(DateTime.DATE_MED); // "Jul 12, 2026"

DateTime.now().toLocaleString(DateTime.DATETIME_SHORT);
```

------------------------------------------------------------------------

## STEP 5 : Use in a Component

```tsx
// components/event-time.tsx
import { DateTime } from "luxon";

export function EventTime({
  isoUtc,
  zone,
}: {
  isoUtc: string;
  zone: string;
}) {
  const dt = DateTime.fromISO(isoUtc, { zone: "utc" }).setZone(zone);
  return (
    <time dateTime={isoUtc}>
      {dt.toLocaleString(DateTime.DATETIME_MED)}
    </time>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Validate Input

```ts
const dt = DateTime.fromISO(userInput);
if (!dt.isValid) {
  throw new Error(`${dt.invalidReason}: ${dt.invalidExplanation}`);
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

Luxon depends on the runtime's Intl/ICU data for zones and locales.
Node on Windows ships full ICU, so results match your Linux servers. If
an older/minimal runtime lacks zone data, install `full-icu`; keep all
stored timestamps in UTC to avoid drift.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Parse with an explicit { zone: "utc" }, then setZone for display
✓ Treat DateTime as immutable — plus/minus return new instances
✓ Check .isValid before using a parsed DateTime
✓ Use toLocaleString(DateTime.PRESET) for localized output
✓ Store UTC ISO strings; convert to user zone only at render
✓ Prefer Luxon when time zones/locales are core to the domain
✓ Use Interval/Duration for ranges instead of manual math
✓ Centralize helpers in lib/date.ts for consistency
```
