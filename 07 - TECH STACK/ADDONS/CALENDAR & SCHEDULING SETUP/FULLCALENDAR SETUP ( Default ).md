# FULLCALENDAR SETUP [ CALENDAR & SCHEDULING ]

------------------------------------------------------------------------

FullCalendar is a feature-rich, framework-agnostic calendar with an
official React wrapper. It renders month/week/day views, drag-to-create,
and event resizing, and fetches events from your API routes.

------------------------------------------------------------------------

## STEP 1 : Install Packages

```bash
pnpm add @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction
```

FullCalendar ships its own CSS via the plugins, so no separate stylesheet
import is required in v6.

------------------------------------------------------------------------

## STEP 2 : Type Your Events

```ts
// lib/events.ts
export interface CalendarEvent {
  id: string;
  title: string;
  start: string; // ISO 8601
  end?: string;
  allDay?: boolean;
}
```

------------------------------------------------------------------------

## STEP 3 : Render The Calendar (Client)

FullCalendar is client-only; mark the component with "use client".

```tsx
"use client";
import FullCalendar from "@fullcalendar/react";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin, { DateSelectArg } from "@fullcalendar/interaction";

export function Scheduler() {
  async function onSelect(info: DateSelectArg) {
    const title = prompt("Event title") ?? "Untitled";
    await fetch("/api/events", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ title, start: info.startStr, end: info.endStr }),
    });
  }

  return (
    <FullCalendar
      plugins={[dayGridPlugin, timeGridPlugin, interactionPlugin]}
      initialView="dayGridMonth"
      headerToolbar={{
        left: "prev,next today",
        center: "title",
        right: "dayGridMonth,timeGridWeek,timeGridDay",
      }}
      selectable
      editable
      events="/api/events"
      select={onSelect}
    />
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Serve Events From A Route Handler

FullCalendar can fetch a URL directly; return an array of events.

```ts
// app/api/events/route.ts
import { NextRequest, NextResponse } from "next/server";
import type { CalendarEvent } from "@/lib/events";

export async function GET() {
  const events: CalendarEvent[] = await db.events.findMany();
  return NextResponse.json(events);
}

export async function POST(req: NextRequest) {
  const body = (await req.json()) as Omit<CalendarEvent, "id">;
  const created = await db.events.create({ data: body });
  return NextResponse.json(created, { status: 201 });
}
```

------------------------------------------------------------------------

## STEP 5 : Data Flow

```text
  <FullCalendar events="/api/events">
        |
        | GET on view change
        v
  Route Handler ---> DB ---> [ { id, title, start, end } ]
        ^
        | POST on select / drag
  user drags to create -> onSelect -> fetch POST -> refetch
```

Wrap `<FullCalendar>` in `dynamic(() => import(...), { ssr: false })` if
you hit hydration warnings from timezone-sensitive rendering.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` runs the same on Windows. Rendering is timezone-driven by the
browser, so a developer machine set to a non-UTC Windows timezone will
show events shifted — store ISO strings in UTC and let the client
localise to avoid confusion across machines.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Store event times as UTC ISO 8601, localise only in the view
✓ Keep the calendar a client component; server data via API routes
✓ Refetch or optimistically update after create/edit/drag
✓ Validate start < end server-side before persisting
✓ Lazy-load with ssr:false if hydration mismatches appear
✓ Debounce rapid view changes that trigger event refetches
✓ Use event IDs from the DB, never array indexes
```
