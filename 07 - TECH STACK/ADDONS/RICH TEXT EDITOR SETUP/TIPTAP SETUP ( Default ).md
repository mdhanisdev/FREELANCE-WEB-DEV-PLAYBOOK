# TIPTAP SETUP [ RICH TEXT EDITOR ]
------------------------------------------------------------------------
Tiptap is a headless, framework-agnostic editor built on ProseMirror.
It ships zero styles, so it fits Tailwind perfectly and gives you full
control over the DOM. This is the default rich text editor for the
stack: batteries-optional, extensible, and SSR-friendly.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add @tiptap/react @tiptap/pm @tiptap/starter-kit
pnpm add @tiptap/extension-placeholder @tiptap/extension-link
```
On Windows the install is identical; if pnpm's global store lives on a
different drive than your project, enable a shared store once with
`pnpm config set store-dir D:\.pnpm-store` to avoid slow cross-drive
hard-link fallbacks.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <Editor/>  (Client Component, "use client")              |
|                                                           |
|   useEditor() ---> Editor instance (ProseMirror state)    |
|        |                                                  |
|        +--> StarterKit  (bold, italic, lists, headings)   |
|        +--> Placeholder (empty-doc hint)                  |
|        +--> Link        (autolink + paste handling)       |
|                                                           |
|   <EditorContent editor={editor} />  --> contentEditable  |
+-----------------------------------------------------------+
        |                          ^
        v                          |
   onUpdate(JSON/HTML) ----> Server Action / API route
```
------------------------------------------------------------------------
## STEP 3 : Build the Editor Component
Tiptap touches the DOM, so it must be a client component.
```tsx
"use client";

import { useEditor, EditorContent } from "@tiptap/react";
import StarterKit from "@tiptap/starter-kit";
import Placeholder from "@tiptap/extension-placeholder";
import Link from "@tiptap/extension-link";

type Props = {
  initialContent?: string;
  onChange?: (html: string) => void;
};

export function Editor({ initialContent = "", onChange }: Props) {
  const editor = useEditor({
    immediatelyRender: false, // required for SSR / App Router
    extensions: [
      StarterKit,
      Placeholder.configure({ placeholder: "Write something..." }),
      Link.configure({ openOnClick: false, autolink: true }),
    ],
    content: initialContent,
    editorProps: {
      attributes: { class: "prose max-w-none focus:outline-none" },
    },
    onUpdate: ({ editor }) => onChange?.(editor.getHTML()),
  });

  if (!editor) return null;
  return <EditorContent editor={editor} />;
}
```
------------------------------------------------------------------------
## STEP 4 : Add a Minimal Toolbar
```tsx
"use client";

import type { Editor } from "@tiptap/react";

export function Toolbar({ editor }: { editor: Editor }) {
  const btn = (active: boolean) =>
    `px-2 py-1 rounded text-sm ${active ? "bg-black text-white" : "bg-gray-100"}`;

  return (
    <div className="flex gap-1 border-b p-2">
      <button
        className={btn(editor.isActive("bold"))}
        onClick={() => editor.chain().focus().toggleBold().run()}
      >
        Bold
      </button>
      <button
        className={btn(editor.isActive("italic"))}
        onClick={() => editor.chain().focus().toggleItalic().run()}
      >
        Italic
      </button>
    </div>
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Style with the Tailwind Typography Plugin
```bash
pnpm add -D @tailwindcss/typography
```
```css
/* globals.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```
The `prose` class on `EditorContent` now renders headings, lists, and
links with sensible defaults you can override per breakpoint.
------------------------------------------------------------------------
## STEP 6 : Persist Content Safely
Store `editor.getJSON()` (portable, diff-friendly) and render HTML on
the server with `generateHTML()` from `@tiptap/html`. Never trust raw
HTML from clients: sanitize before persisting or rendering.
```ts
import { generateHTML } from "@tiptap/html";
import StarterKit from "@tiptap/starter-kit";

export function renderDoc(json: object): string {
  return generateHTML(json, [StarterKit]);
}
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Set immediatelyRender:false to avoid App Router hydration warnings
✓ Persist JSON, not HTML — it is portable and safe to re-render
✓ Sanitize any HTML before saving or displaying it
✓ Keep the editor inside a "use client" boundary, isolated small
✓ Debounce onUpdate before firing Server Actions on every keystroke
✓ Use @tailwindcss/typography for consistent content styling
✓ Register only the extensions you need to keep the bundle lean
✓ Destroy is automatic via useEditor — never new Editor() in effects
```
