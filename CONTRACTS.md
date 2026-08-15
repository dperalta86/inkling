# inkling — CONTRACTS.md

> Status: frozen for Day-0 / production architecture v1.
> Later changes require explicit approval from every consuming track of the affected contract, plus the `contract-change` label on the PR.
> This document defines the minimum accepted interface between modules. The real implementation may vary internally, but the public signature stays fixed.

---

## Repo conventions

- Rust commands: `snake_case` on the backend, `#[tauri::command]`.
- TypeScript wrappers: `camelCase` on the frontend.
- The editor uses ProseMirror with a centralized schema.
- The file on disk stays plain Markdown; no editor-specific metadata is saved.
- If a change breaks a contract already frozen, it doesn't get merged without a cross decision from all consumers of that contract.

---

## Contract 1 — ProseMirror Schema (Track B → consumed by: C, D)

Target file:

```typescript
// src/lib/editor/schema.ts
export const schema: Schema;
```

Public schema definition:

```typescript
// Nodes
// - doc
// - paragraph
// - heading(attrs: { level: 1 | 2 | 3 | 4 | 5 | 6 })
// - bullet_list
// - list_item
// - code_block(attrs: { params?: string })
// - horizontal_rule
// - blockquote
// - text
// - hard_break

// Marks
// - strong
// - em
// - code
// - link(attrs: { href: string; title?: string })
// - mdSymbol(attrs: { kind: 'open' | 'close' })
```

Contract rules:

- `mdSymbol` is a mark used to tag the visible Markdown syntax delimiters in the editor (`**`, `*`, `` ` ``, `#`, `-`, etc.).
- `mdSymbol.kind` distinguishes opening/closing so the dynamic-focus plugin can render the right marker and apply left- or right-side styling to the token.
- `link` requires `href`; `title` is optional and must be serialized as an HTML-safe attribute if present.
- `code_block` accepts an optional `params` for language metadata; if unused, it's serialized without the attribute.
- The schema centralizes the full set of nodes and marks for Markdown editing; any plugin or UI must consume it as the single source of truth.

### Closed decision

- `mdSymbol` requires the `kind: 'open' | 'close'` attribute. A version without that field is not accepted.
- `focus-decor.ts` must determine whether a token is opening or closing by reading that attribute.

---

## Contract 2 — Parser and post-processor (Track B → consumed by: D via integration, E via tests)

Target files:

```typescript
// src/lib/markdown/parser.ts
export function parseMarkdown(md: string): Node;

// src/lib/markdown/post-processor.ts
export function addMdSymbolMarks(doc: Node): Node;
```

Semantics:

- `parseMarkdown(md: string): Node` takes a Markdown string and returns a ProseMirror `Node` compatible with the `schema` from Contract 1.
- `addMdSymbolMarks(doc: Node): Node` takes the already-parsed document and adds `mdSymbol` marks over the Markdown delimiters that require visual tracking/focus.
- The post-processor must be idempotent: applying it twice to the same document must not duplicate marks or transform already-valid content.
- The parser must preserve the exact textual content, except for the expected semantic changes from Markdown to the ProseMirror AST.

Interoperability requirements:

- The output of `parseMarkdown` must respect the `schema` from Contract 1.
- The post-processor must run after parsing and before rendering the editor view.
- Round-trip tests must validate that `serializeMarkdown(parseMarkdown(md))` doesn't change the document's semantic content, except for whitespace normalization differences allowed by Markdown.

> **Note on consumer ownership:** the actual caller of these functions in code is Track D (`Editor.svelte`, when opening a file into the editor), not Track A. Track A only touches raw strings via the Tauri commands in Contract 4/5 — it never imports TS code. Listing D as the code-level consumer here avoids giving A veto power over a signature it doesn't call.

---

## Contract 3 — Serializer and clipboard (Track B → consumed by: D)

Target files:

```typescript
// src/lib/markdown/serializer.ts
export function serializeMarkdown(doc: Node): string;

// src/lib/markdown/clipboard.ts
export function serializeSlice(slice: Slice): string;
```

Semantics:

- `serializeMarkdown(doc: Node): string` exports a ProseMirror document to plain Markdown, with no editor metadata or hidden text.
- `serializeSlice(slice: Slice): string` serializes a content fragment (e.g. copy/paste) to plain Markdown.
- The serializer must treat `mdSymbol` as a visual mark and must not serialize it as real document content.
- The output must read as Markdown, not as an internal ProseMirror representation.

Integration rule:

- The serializer is used both to save to the file and to copy selected content to the clipboard.
- The model represents the same Markdown document as the file on disk; there is no parallel storage format.

> **Note on consumer ownership:** as with Contract 2, the actual code-level caller is Track D — it calls `serializeMarkdown`/`serializeSlice` and then hands the resulting string to Track A's `write_file`/clipboard APIs. Track A receives a string, it doesn't call into this module.

---

## Contract 4 — Tauri commands (Track A → consumed by: D, B via integration)

Target files:

```rust
// src-tauri/src/fs.rs
#[tauri::command]
async fn read_file(path: String) -> Result<String, String>;

#[tauri::command]
async fn write_file(path: String, content: String) -> Result<(), String>;

#[tauri::command]
async fn delete_file(path: String) -> Result<(), String>;

// src-tauri/src/dialog.rs
#[tauri::command]
async fn pick_file() -> Result<Option<String>, String>;

#[tauri::command]
async fn save_dialog() -> Result<Option<String>, String>;

// src-tauri/src/autosave.rs
#[tauri::command]
async fn write_swap(path: String, content: String) -> Result<(), String>;

#[tauri::command]
async fn read_swap(path: String) -> Result<Option<String>, String>;

#[tauri::command]
async fn delete_swap(path: String) -> Result<(), String>;

// src-tauri/src/watch.rs
#[tauri::command]
async fn watch_file(path: String) -> Result<(), String>;
```

Semantics:

- `read_file(path)` returns the file's content as a UTF-8 string. If the file doesn't exist or can't be read, it returns `Err(String)` with a readable message.
- `write_file(path, content)` writes the full file content to disk; no incremental merge.
- `delete_file(path)` deletes the file if it exists, returning an error if it can't.
- `pick_file()` returns the path selected by the user, or `None` if canceled.
- `save_dialog()` returns a destination path, or `None` if canceled.
- `write_swap(path, content)` writes a `.swp` swap file associated with the working file.
- `read_swap(path)` attempts to recover the swap content for the file; returns `None` if it doesn't exist.
- `delete_swap(path)` deletes the associated swap file.
- `watch_file(path)` registers the path with the filesystem watcher; the backend must emit the `file_changed_externally` event when it detects changes to that file.

Event emitted by the watcher:

```typescript
// payload of the event emitted by the backend
{
  path: string;
  kind: 'modified' | 'deleted';
}
```

### Closed decision

- `file_changed_externally` includes `kind` to distinguish external modification from deletion.
- `ReloadBanner.svelte` must react to `kind === 'deleted'` and `kind === 'modified'` with distinct behavior.

---

## Contract 5 — TypeScript `invoke` wrapper (Track A/D shared)

Target file:

```typescript
// src/lib/tauri/commands.ts
export function readFile(path: string): Promise<string>;
export function writeFile(path: string, content: string): Promise<void>;
export function deleteFile(path: string): Promise<void>;
export function pickFile(): Promise<string | null>;
export function saveDialog(): Promise<string | null>;
export function writeSwap(path: string, content: string): Promise<void>;
export function readSwap(path: string): Promise<string | null>;
export function deleteSwap(path: string): Promise<void>;
export function watchFile(path: string): Promise<void>;
```

Rules:

- Exported TS names use `camelCase`.
- Each function maps 1:1 to a Rust Tauri command with the same conceptual name.
- Functions must wrap `invoke` and convert the result to the expected TS types.
- The wrapper must not swallow errors: they must propagate as rejected promises or runtime errors that the UI handles.

---

## Contract 6 — ProseMirror plugins (Track C → consumed by: D)

Target files:

```typescript
// src/lib/editor/gutter.ts
export function gutterPlugin(): Plugin;
export const gutterLines: Writable<GutterLine[]>;

// src/lib/editor/focus-decor.ts
export function focusDecorPlugin(): Plugin;
export const FOCUS_MARGIN: number;

// src/lib/editor/input-rules.ts
export function inputRulesPlugin(): Plugin;

// src/lib/editor/keymap.ts
export function customKeymapPlugin(): Plugin;
```

Public definition of `GutterLine`:

```typescript
export type GutterLine = {
  level: number;
  from: number;
  to: number;
  type: 'heading' | 'list' | 'code';
};
```

Rules:

- `gutterLines` is a writable Svelte store holding the set of structure markers to paint in the gutter.
- `from` and `to` are ProseMirror document offsets (document positions).
- `type` classifies the line as `heading`, `list`, or `code`.
- `level` represents the visual hierarchy for the gutter: for headings, the `#` count; for lists, the nesting depth; for code blocks, the block's depth/structure (if applicable).
- `FOCUS_MARGIN` defaults to `1` and is configurable at build time or via env; it stays as an exported constant so other frontend layers can read it.
- `focusDecorPlugin` must interact with `mdSymbol` and cursor position to dim or highlight delimiters based on proximity.

### Closed decision

- The exact shape of `GutterLine` is fixed as above and must be respected by `Gutter.svelte` and any plugin that produces gutter lines.

---

## Contract 7 — Svelte components (Track D → consumed by: E for integration tests)

Public components and props/stores:

```typescript
// src/lib/components/Editor.svelte
export const editorView: Writable<EditorView>;

// src/lib/components/Gutter.svelte
// Reads `gutterLines` from Contract 6

// src/lib/components/Toolbar.svelte
// Reads `editorView` and runs ProseMirror commands

// src/lib/components/StatusBar.svelte
export type DocState = {
  saved: boolean;
  lines: number;
  path: string;
};

// src/lib/components/ReloadBanner.svelte
export const visible: Writable<boolean>;
```

Rules:

- `Editor.svelte` exposes `editorView` via context so frontend components can run commands, transformations, or inspect the current editor.
- `Gutter.svelte` consumes `gutterLines` and renders the document's structure information.
- `Toolbar.svelte` must not depend on `document` or on internal DOM structure outside of `editorView`.
- `StatusBar.svelte` represents the current document's state with the fields `saved`, `lines`, and `path`.
- `ReloadBanner.svelte` activates when the watcher emits `file_changed_externally` and must hide once the user confirms or resolves the sync conflict.

---

## Contract 8 — File save/sync state (Track D → consumed by: A, E)

Owner: **Track D**, since it's the layer that orchestrates the raw file I/O from Track A and the document edits tracked by Track B's ProseMirror state into a single source of truth for "is this document saved."

Target file:

```typescript
// src/lib/stores/file-sync.ts
export const fileSyncState: Writable<FileSyncState>;

export type FileSyncState = {
  saved: boolean;
  dirty: boolean;
  path: string | null;
  lastSavedAt?: number;
};
```

Semantics:

- `saved === true` means the editor's current content matches what's persisted to the file.
- `dirty === true` means the editor has pending unsaved changes.
- `path` points to the current file's path; if no document is open, it must be `null`.
- `lastSavedAt` is optional and used for UI/logs; not required for core logic.
- The external watcher must notify changes without rewriting the file from the local edit; any re-render or banner must have a clear origin.
- `fileSyncState` is the **only** place that computes `saved`/`dirty` — no other module keeps its own parallel copy of this state. Autosave (Track A) and the editor's transaction stream (Track B) feed into it, they don't replace it.

> **Why this contract needed an explicit owner:** unlike the others, this one originally had no assigned track or file path — just a type definition. Without a single owner, two tracks could end up computing `dirty`/`saved` independently and drift out of sync. Track D owns it because it's the only layer that already talks to both A (autosave/file I/O) and B (document transactions).

---

## Post-freeze change rules

1. Any change that modifies a signature published in this document must be reviewed as a `contract-change`.
2. The `contract-change` label requires approval from every track that consumes the modified contract.
3. Changes to names, types, payloads, routes, or public events require updating this documentation before merge.
4. If a change breaks compatibility with an existing consumer, it must be explicitly documented in the changelog section.

---

## Post-freeze changelog

| Date | Affected contract | Change | Approved by |
|---|---|---|---|
| — | — | — | — |

> Filled in only when an already-frozen contract needs modification. Not used for internal implementation changes with no public-interface impact.

---

## Consumers per contract

- Contract 1 (Schema): Track C, Track D
- Contract 2 (Parser/Post-processor): Track D (integration), Track E (tests)
- Contract 3 (Serializer/Clipboard): Track D
- Contract 4 (Tauri commands/watch): Track D, Track B (integration)
- Contract 5 (TS wrapper): Track A, Track D
- Contract 6 (Plugins): Track D
- Contract 7 (Svelte components): Track E
- Contract 8 (File sync state): Track A, Track E

---

## Session close / final decision

This document replaces the Day-0 temporary template and stands as the team's reference version. Each module's implementation can proceed in parallel, but every integration must respect this public contract. Any later change must go back through cross-review — a "local fix" that breaks the shared interface is not accepted.