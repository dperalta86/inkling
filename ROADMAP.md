# Roadmap

No dates — this is a map of direction, not a delivery commitment. The order of versions does matter: each one builds on the interfaces frozen by the previous one (see `CONTRACTS.md`), so steps aren't skipped.

## v0.1 — MVP · *in development*

The goal of this version is a single thing: validate that writing in inkling *feels* right, not just that it looks right.

- Markdown editor with soft-render and dynamic focus (bold, italic, inline code, links, lists).
- Document structure gutter.
- Basic input rules (`## `, `- `, `**x**`, etc.).
- Autosave + explicit save.
- Detection of external file changes.
- Light/dark mode.
- Linux only.

## v0.2 — Full formatting

- Code blocks with syntax highlighting.
- Blockquotes and horizontal rules with dedicated rendering.
- Images (drag & drop).
- Front matter (YAML) recognition.
- PDF export.

## v0.3 — Local AI assistance

- Integration with locally-running models (no dependency on a cloud service).
- Select text → rewrite suggestion → inline diff, accept or reject.

## v0.4 — Maturity

- WYSIWYG table editing.
- Search and replace.
- Document outline navigator.
- Multi-tab support.
- Plugin API.
- Windows and macOS, if there's enough demand to justify it.

---

Missing something you'd like to see? Open an issue — the roadmap adjusts with real usage, it isn't a closed list.