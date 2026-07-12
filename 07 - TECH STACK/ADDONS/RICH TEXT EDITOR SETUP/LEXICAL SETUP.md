# LEXICAL SETUP [ RICH TEXT EDITOR ]
------------------------------------------------------------------------
Lexical is Meta's extensible text editor framework. It is faster and
lighter than most alternatives, uses an immutable node tree, and drives
a first-class React binding. Reach for it when you need fine-grained
control over the editor state model and predictable performance.
------------------------------------------------------------------------
## STEP 1 : Install Dependencies
```bash
pnpm add lexical @lexical/react
pnpm add @lexical/rich-text @lexical/utils
```
Windows note: Lexical is pure TypeScript with no native bindings, so
installs never trigger node-gyp — no Visual Studio Build Tools needed.
------------------------------------------------------------------------
## STEP 2 : Architecture Overview
```text
+-----------------------------------------------------------+
|  <LexicalComposer initialConfig>                          |
|        |                                                  |
|        +-- <RichTextPlugin/>   (contentEditable surface)  |
|        +-- <HistoryPlugin/>    (undo / redo)              |
|        +-- <OnChangePlugin/>   (editorState listener)     |
|                                                           |
|   EditorState (immutable tree of Nodes)                   |
|        |                                                  |
|        v  editorState.read(() => $generateHtml...)        |
|   serialized JSON / HTML  --> Server Action               |
+-----------------------------------------------------------+
```
------------------------------------------------------------------------
## STEP 3 : Configure the Composer
```tsx
"use client";

import { LexicalComposer } from "@lexical/react/LexicalComposer";
import { RichTextPlugin } from "@lexical/react/LexicalRichTextPlugin";
import { ContentEditable } from "@lexical/react/LexicalContentEditable";
import { HistoryPlugin } from "@lexical/react/LexicalHistoryPlugin";
import { LexicalErrorBoundary } from "@lexical/react/LexicalErrorBoundary";
import { HeadingNode, QuoteNode } from "@lexical/rich-text";

const initialConfig = {
  namespace: "app-editor",
  nodes: [HeadingNode, QuoteNode],
  onError: (e: Error) => console.error(e),
  theme: {
    paragraph: "mb-2",
    heading: { h1: "text-2xl font-bold", h2: "text-xl font-semibold" },
  },
};

export function Editor() {
  return (
    <LexicalComposer initialConfig={initialConfig}>
      <div className="rounded border p-3">
        <RichTextPlugin
          contentEditable={
            <ContentEditable className="min-h-40 focus:outline-none" />
          }
          placeholder={
            <p className="text-gray-400">Start writing...</p>
          }
          ErrorBoundary={LexicalErrorBoundary}
        />
        <HistoryPlugin />
      </div>
    </LexicalComposer>
  );
}
```
------------------------------------------------------------------------
## STEP 4 : Read State on Change
```tsx
"use client";

import { OnChangePlugin } from "@lexical/react/LexicalOnChangePlugin";
import type { EditorState } from "lexical";

export function ChangeListener({
  onChange,
}: {
  onChange: (json: string) => void;
}) {
  return (
    <OnChangePlugin
      onChange={(state: EditorState) =>
        onChange(JSON.stringify(state.toJSON()))
      }
    />
  );
}
```
------------------------------------------------------------------------
## STEP 5 : Add a Toolbar via the Editor Context
```tsx
"use client";

import { useLexicalComposerContext } from
  "@lexical/react/LexicalComposerContext";
import { FORMAT_TEXT_COMMAND } from "lexical";

export function Toolbar() {
  const [editor] = useLexicalComposerContext();
  return (
    <button
      className="rounded bg-gray-100 px-2 py-1 text-sm"
      onClick={() => editor.dispatchCommand(FORMAT_TEXT_COMMAND, "bold")}
    >
      Bold
    </button>
  );
}
```
------------------------------------------------------------------------
## STEP 6 : Serialize for Storage
Persist `editorState.toJSON()`. To render read-only content later, set
`editable: false` in the config and hydrate with `setEditorState`.
```ts
const state = editor.parseEditorState(savedJsonString);
editor.setEditorState(state);
```
------------------------------------------------------------------------
## BEST PRACTICES
```text
✓ Register every custom node in initialConfig.nodes up front
✓ Mutate state only inside editor.update() transactions
✓ Read state inside editorState.read() — never touch DOM directly
✓ Keep the whole tree inside a "use client" boundary
✓ Persist toJSON() output; it round-trips losslessly
✓ Use LexicalErrorBoundary so a bad node never crashes the page
✓ Debounce OnChangePlugin before hitting the network
✓ Set editable:false for read-only rendering instead of a new lib
```
