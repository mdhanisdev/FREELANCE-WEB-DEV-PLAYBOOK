# BLOCKNOTE SETUP [ RICH TEXT EDITOR ]
------------------------------------------------------------------------
BlockNote is a Notion-style block editor built on top of Tiptap and
ProseMirror. It ships a polished UI out of the box — slash menu, drag
handles, formatting toolbar — so you trade some headless flexibility
for speed of delivery. Ideal for docs, wikis, and CMS body fields.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add @blocknote/core @blocknote/react @blocknote/mantine
```
Windows note: BlockNote pulls in Mantine styles; if your editor's
watcher misses CSS changes on Windows, set `WATCHPACK_POLLING=true` in
`.env.local` to force polling-based file watching.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <BlockNoteView editor>   (pre-built Mantine UI)          |
|        |                                                  |
|        +-- Slash Menu        ("/" inserts blocks)         |
|        +-- Drag Handles      (reorder blocks)             |
|        +-- Formatting Toolbar(select-to-format)           |
|                                                           |
|   useCreateBlockNote() ---> editor (Block[] document)     |
|        |                                                  |
|        v  editor.document  (array of typed blocks)        |
|   JSON blocks --> Server Action --> DB                    |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Build the Editor Component
```tsx
"use client";

import "@blocknote/mantine/style.css";
import { useCreateBlockNote } from "@blocknote/react";
import { BlockNoteView } from "@blocknote/mantine";
import type { Block } from "@blocknote/core";

type Props = {
  initialContent?: Block[];
  onChange?: (blocks: Block[]) => void;
};

export function Editor({ initialContent, onChange }: Props) {
  const editor = useCreateBlockNote({ initialContent });

  return (
    <BlockNoteView
      editor={editor}
      onChange={() => onChange?.(editor.document)}
      className="rounded border"
    />
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Match Your Theme
BlockNote reads light/dark from a `theme` prop or the `data-color-scheme`
attribute. Wire it to your Tailwind dark mode toggle.
```tsx
"use client";

import { BlockNoteView } from "@blocknote/mantine";

export function ThemedView({ editor, dark }: { editor: any; dark: boolean }) {
  return <BlockNoteView editor={editor} theme={dark ? "dark" : "light"} />;
}
```
------------------------------------------------------------------------
## STEP 5 : Export to HTML or Markdown
```tsx
"use client";

export async function exportDoc(editor: any) {
  const html = await editor.blocksToHTMLLossy(editor.document);
  const md = await editor.blocksToMarkdownLossy(editor.document);
  return { html, md };
}
```
Note these exporters are lossy for custom blocks — persist the raw
`Block[]` JSON as the source of truth and treat HTML/MD as views.
------------------------------------------------------------------------
## STEP 6 : Server Rendering
BlockNote is client-only. For static rendering, store the block JSON and
convert with the server-safe exporter package at request time.
```bash
pnpm add @blocknote/server-util
```
```ts
import { ServerBlockNoteEditor } from "@blocknote/server-util";

export async function toHtml(blocks: object[]) {
  const editor = ServerBlockNoteEditor.create();
  return editor.blocksToFullHTML(blocks as any);
}
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Import the Mantine style.css exactly once, in the editor module
✓ Persist Block[] JSON — HTML/Markdown exports are lossy
✓ Keep BlockNoteView inside a "use client" boundary
✓ Use @blocknote/server-util for SSR / static HTML output
✓ Wire theme prop to your app's dark-mode source of truth
✓ Debounce onChange before sending editor.document to the server
✓ Define custom blocks with the schema API, not DOM hacks
✓ Pin the three @blocknote/* packages to one matching version
```
