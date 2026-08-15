<div align="center">

# inkling

**Markdown, without the noise.**

[![status](https://img.shields.io/badge/status-early%20development-blue)]()
[![platform](https://img.shields.io/badge/platform-linux-informational)]()
[![license](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

[English](./README.md) | [Español](./README.es.md)

</div>

<!-- TODO: add a screenshot or gif of the editor here once there's something stable to show -->

---

inkling is a lightweight desktop editor for Markdown. It doesn't hide the syntax like a traditional WYSIWYG, nor does it force you to read it at full volume like a plain text editor: it dims it. The symbols are still there — `**`, `#`, `` ` `` — but they step back visually until the cursor needs them.

The file on disk is plain Markdown at all times. inkling doesn't add its own metadata or invent a format: it's a different way of looking at the same file.

## Why

Markdown editing tools usually force a choice: either you see the symbols and lose reading flow, or you don't see them and lose structural transparency. inkling bets that you shouldn't have to choose — syntax can be present without being in the foreground.

## What this looks like in practice

- **Soft-render with dynamic focus** — delimiters dim when the cursor is far away and return to normal size when the cursor touches them. They never stop being editable text.
- **Structure gutter** — a side margin shows the document's hierarchy (headings, lists, code blocks) without needing to read the symbols.
- **Input rules** — type `## ` and it becomes a heading, `- ` and it becomes a list, without breaking your writing flow.
- **Local-first** — the file lives on your filesystem. No account, no forced sync, no lock-in.
- **Lightweight** — built on Tauri, not Electron. Small binary, low RAM footprint.

## Project status

In active development. The full architecture and roadmap live in [`ROADMAP.md`](./ROADMAP.md). For now, the focus is Linux — Windows and macOS are a future possibility, not a v0.1 commitment.

## Installation

*(Pending first release — the installation script will be documented here once there's a binary to distribute.)*

## Contributing

Before opening a PR, check [`CONTRIBUTING.md`](./CONTRIBUTING.md) — in particular the DCO section (commit sign-off), which is a requirement, not a suggestion.

The technical contracts between modules (schema, Tauri commands, plugins) are frozen in [`CONTRACTS.md`](./CONTRACTS.md). If your change touches one of those contracts, the PR will flag it explicitly.

## License

[MIT](./LICENSE)