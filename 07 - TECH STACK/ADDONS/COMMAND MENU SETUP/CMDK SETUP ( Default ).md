# CMDK SETUP [ COMMAND MENU ]
------------------------------------------------------------------------

`cmdk` is a fast, composable command palette for React — the ⌘K menu
you see in Linear, Vercel, and Raycast-style UIs. shadcn/ui wraps it as
the `Command` component, giving you keyboard nav, fuzzy filtering, and
a dialog variant out of the box.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add cmdk
pnpm dlx shadcn@latest add command dialog
```

The shadcn `command` component is a styled cmdk; `dialog` powers the
modal ⌘K overlay.

------------------------------------------------------------------------

## STEP 2 : Anatomy

```text
+------------------------------------------------------------+
|  CommandDialog (⌘K opens)                                  |
|    +----------------------------------------------------+  |
|    | CommandInput   (fuzzy filter)                      |  |
|    | CommandList                                        |  |
|    |   CommandEmpty  "No results"                       |  |
|    |   CommandGroup "Navigation"                        |  |
|    |     CommandItem -> onSelect(action)                |  |
|    |   CommandGroup "Actions"                           |  |
|    +----------------------------------------------------+  |
+------------------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 3 : Global Command Menu

```tsx
// components/command-menu.tsx
"use client";
import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import {
  CommandDialog,
  CommandInput,
  CommandList,
  CommandEmpty,
  CommandGroup,
  CommandItem,
} from "@/components/ui/command";

export function CommandMenu() {
  const [open, setOpen] = useState(false);
  const router = useRouter();

  useEffect(() => {
    const down = (e: KeyboardEvent) => {
      if (e.key === "k" && (e.metaKey || e.ctrlKey)) {
        e.preventDefault();
        setOpen((o) => !o);
      }
    };
    document.addEventListener("keydown", down);
    return () => document.removeEventListener("keydown", down);
  }, []);

  const go = (href: string) => {
    setOpen(false);
    router.push(href);
  };

  return (
    <CommandDialog open={open} onOpenChange={setOpen}>
      <CommandInput placeholder="Type a command or search…" />
      <CommandList>
        <CommandEmpty>No results found.</CommandEmpty>
        <CommandGroup heading="Navigation">
          <CommandItem onSelect={() => go("/dashboard")}>
            Dashboard
          </CommandItem>
          <CommandItem onSelect={() => go("/settings")}>
            Settings
          </CommandItem>
        </CommandGroup>
      </CommandList>
    </CommandDialog>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Mount in the Root Layout

```tsx
// app/layout.tsx
import { CommandMenu } from "@/components/command-menu";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <CommandMenu />
      </body>
    </html>
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Show the Shortcut Hint

```tsx
// components/kbd-hint.tsx
export function KbdHint() {
  return (
    <kbd className="rounded border px-1.5 py-0.5 text-xs">
      ⌘ K
    </kbd>
  );
}
```

------------------------------------------------------------------------

## WINDOWS NOTES

Windows keyboards have no ⌘ key, so the handler checks `e.ctrlKey`
alongside `e.metaKey` — Ctrl+K works on Windows, ⌘K on macOS. Label the
hint "Ctrl K" for Windows users via a platform check if desired.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Check both metaKey and ctrlKey so ⌘K/Ctrl+K both open the menu
✓ Always e.preventDefault() to stop the browser's default ⌘K
✓ Mount CommandMenu once in the root layout, not per page
✓ Group items with CommandGroup headings for scannability
✓ Close the dialog before navigating to avoid flashes
✓ Provide CommandEmpty text so empty searches aren't blank
✓ Keep items keyboard-first — cmdk handles arrow-key nav for you
✓ Lazy-load heavy actions; keep the item list cheap to render
```
