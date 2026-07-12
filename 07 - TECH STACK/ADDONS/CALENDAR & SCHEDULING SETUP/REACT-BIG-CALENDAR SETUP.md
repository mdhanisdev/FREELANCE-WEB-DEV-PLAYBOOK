# REACT BIG CALENDAR SETUP [ CALENDAR & SCHEDULING ]

------------------------------------------------------------------------

react-big-calendar is a lightweight, Google-Calendar-style component that
renders events over month/week/day/agenda views. It needs a date
localizer (date-fns) and controlled event state.

------------------------------------------------------------------------

## STEP 1 : Install Packages

```bash
pnpm add react-big-calendar date-fns
pnpm add -D @types/react-big-calendar
```

The component ships a stylesheet you must import once.

------------------------------------------------------------------------

## STEP 2 : Configure The Localizer

```ts
// lib/localizer.ts
import { dateFnsLocalizer } from "react-big-calendar";
import { format, parse, startOfWeek, getDay } from "date-fns";
import { enUS } from "date-fns/locale";

export const localizer = dateFnsLocalizer({
  format,
  parse,
  startOfWeek,
  getDay,
  locales: { "en-US": enUS },
});
```

------------------------------------------------------------------------

## STEP 3 : Render The Calendar (Client)

```tsx
"use client";
import { Calendar, Views, type Event } from "react-big-calendar";
import { useState, useEffect } from "react";
import { localizer } from "@/lib/localizer";
import "react-big-calendar/lib/css/react-big-calendar.css";

export function Scheduler() {
  const [events, setEvents] = useState<Event[]>([]);

  useEffect(() => {
    fetch("/api/events")
      .then((r) => r.json())
      .then((rows) =>
        setEvents(
          rows.map((e: any) => ({
            title: e.title,
            start: new Date(e.start),
            end: new Date(e.end),
          })),
        ),
      );
  }, []);

  return (
    <Calendar
      localizer={localizer}
      events={events}
      defaultView={Views.WEEK}
      views={[Views.MONTH, Views.WEEK, Views.DAY, Views.AGENDA]}
      style={{ height: 600 }}
      selectable
      onSelectSlot={(slot) =>
        console.log("create between", slot.start, slot.end)
      }
    />
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Serve Events

```ts
// app/api/events/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const rows = await db.events.findMany();
  return NextResponse.json(rows); // { title, start, end } ISO strings
}
```

------------------------------------------------------------------------

## STEP 5 : Rendering Flow

```text
  fetch /api/events
        |
        v
  map ISO -> Date objects (start, end)
        |
        v
  <Calendar events={...} localizer={dateFns} />
        |
  onSelectSlot -> open create modal -> POST -> refresh state
```

react-big-calendar expects real `Date` objects, not ISO strings — the
mapping step is required, unlike FullCalendar which accepts a URL.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` is unaffected by OS. Dates are constructed with the browser
timezone, so a Windows machine on a local timezone will display shifted
times unless you normalise to UTC. Import the CSS exactly once to avoid
duplicated styles in the bundle.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Convert ISO strings to Date objects before passing to Calendar
✓ Import the bundled CSS a single time, high in the tree
✓ Keep events in controlled state for optimistic updates
✓ Store times as UTC and localise via the date-fns localizer
✓ Provide MONTH/WEEK/DAY/AGENDA and remember the user's choice
✓ Constrain container height; the grid needs an explicit size
✓ Validate slot selections server-side before persisting
```
