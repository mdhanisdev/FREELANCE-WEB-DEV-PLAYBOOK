# DATE-FNS SETUP [ DATE HANDLING ]
------------------------------------------------------------------------

date-fns is a modular date utility library. Every function is a pure,
tree-shakeable import that operates on native `Date` objects — no
custom wrapper type. Pair it with the built-in `Intl` API for
locale-aware formatting and you cover most app needs with a tiny bundle.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add date-fns
pnpm add date-fns-tz   # optional: time-zone helpers
```

------------------------------------------------------------------------

## STEP 2 : Import Only What You Use

```text
+------------------------------------------------------------+
|  import { format } from "date-fns"                         |
|          |                                                 |
|          v  (tree-shaken - only format() bundled)          |
|  format(date, "yyyy-MM-dd")   -> "2026-07-12"              |
|                                                            |
|  Intl.DateTimeFormat  -> locale-aware display strings      |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : A Date Utils Module

```ts
// lib/date.ts
import { format, formatDistanceToNow, addDays, isAfter } from "date-fns";

export const iso = (d: Date) => format(d, "yyyy-MM-dd");

export const relative = (d: Date) =>
  formatDistanceToNow(d, { addSuffix: true }); // "3 days ago"

export const dueDate = (start: Date, days: number) => addDays(start, days);

export const isOverdue = (due: Date) => isAfter(new Date(), due);
```

------------------------------------------------------------------------

## STEP 4 : Intl For Display Formatting

Use date-fns for math/parsing and `Intl` for user-facing, localized
output — it needs no extra bundle and respects the user's locale:

```ts
// lib/format-date.ts
export function formatDate(
  date: Date,
  locale = "en-US"
): string {
  return new Intl.DateTimeFormat(locale, {
    dateStyle: "medium",
    timeStyle: "short",
  }).format(date);
}

export function formatMoneyDate(date: Date) {
  // e.g. "Jul 12, 2026"
  return new Intl.DateTimeFormat("en-US", { dateStyle: "medium" }).format(
    date
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Use in a Component

```tsx
// components/due-badge.tsx
import { relative, isOverdue } from "@/lib/date";

export function DueBadge({ due }: { due: Date }) {
  const overdue = isOverdue(due);
  return (
    <span className={overdue ? "text-red-600" : "text-muted-foreground"}>
      {relative(due)}
    </span>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Time Zones (date-fns-tz)

```ts
import { formatInTimeZone } from "date-fns-tz";

const utc = new Date("2026-07-12T18:00:00Z");
formatInTimeZone(utc, "America/New_York", "yyyy-MM-dd HH:mm zzz");
```

------------------------------------------------------------------------

## WINDOWS NOTES

Pure JS — no native deps. `Intl` uses the Node/V8 ICU data bundled with
your runtime, so locale output is identical on Windows and Linux CI. To
avoid server/client mismatch, store dates in UTC and format on render.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Import named functions only — never `import * as` (kills tree-shaking)
✓ Store and transport dates in UTC / ISO 8601, format at the edge
✓ Use date-fns for math+parsing, Intl for localized display
✓ Centralize helpers in lib/date.ts for one source of truth
✓ Pass explicit locale to Intl to avoid server/client drift
✓ Use date-fns-tz for zone conversions; don't hand-roll offsets
✓ Prefer formatDistanceToNow for relative "x ago" labels
✓ Keep Date objects immutable — date-fns never mutates inputs
```
