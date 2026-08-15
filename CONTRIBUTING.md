# Contributing to inkling

## Developer Certificate of Origin (DCO)

Every commit must be signed off, certifying that the code you're contributing is yours and that you're licensing it to the project under the repo's MIT license.

```bash
git commit -s -m "feat(b): markdown parser to ProseMirror Node"
```

This automatically appends a line to the end of the commit message:

```
Signed-off-by: Your Name <your-email@example.com>
```

If you forgot to sign a commit you already made:

```bash
git commit --amend -s --no-edit
```

Set this up once so you don't have to remember it every time:

```bash
git config alias.cs "commit -s"
```

**Why this exists:** it keeps the door open for the future (e.g. dual-licensing, a commercial version) without having to track down and ask retroactive permission from everyone who contributed code. It's free to set up from commit 1, expensive to set up later.

## Workflow

- Repo: monorepo, trunk-based. `main` is protected (PR + 1 review + green CI).
- Branches: `track/US-XXX-short-description` (e.g. `b/US-007-schema-mdsymbol`, `c/US-007-gutter-plugin`, `a/US-003-autosave-swap`). Always include the story ID for traceability back to `REQUIREMENTS.md`.
- Commits: `type(track): description` — types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.
- Every PR references an issue (`Closes #N`).
- If your PR modifies a signature already frozen in `CONTRACTS.md`, add the `contract-change` label — it requires approval from every track that consumes that contract (see `CONTRACTS.md` for the list of consumers per contract).

## Definition of Done

- [ ] Implements the exact signature frozen in `CONTRACTS.md` (changing it requires cross-track approval, see above).
- [ ] No `any` in TypeScript / no `unwrap()` or `panic!` in Rust outside of tests.
- [ ] Commits are signed off with DCO (`git commit -s`).
- [ ] If the PR touches a module listed under Tests below, edge-case tests are included and CI is green.
- [ ] PR is linked to a GitHub issue.
- [ ] At least 1 approved review — from another track if possible, to catch integration issues early.

## Tests

Three modules have a mandatory test gate in CI — a PR touching them won't merge without tests covering these edge cases. Everything else (UI components, individual input rules, keymap) is optional at the reviewer's discretion, since bugs there are low-impact and immediately visible in normal use.

- **Markdown parser/serializer** (round-trip: `parse(serialize(doc)) === doc`): empty documents, delimiters at the very edge of the document, nested/overlapping marks, mixed line endings (CRLF/LF), unicode/emoji inside marks.
- **Autosave** (`autosave.rs`): a race between a `.swp` write and a concurrent `Ctrl+S`, `.swp` recovery after a crash, write failures (disk full, permissions).
- **File watcher** (`watch.rs`): must not self-trigger on inkling's own writes, must debounce rapid successive external changes, and must keep watching a path across a temp-file-plus-rename replacement (the pattern editors like Vim use).