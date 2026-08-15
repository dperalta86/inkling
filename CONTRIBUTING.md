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
- Branches: `track/short-description` (e.g. `b/schema-mdsymbol`, `c/gutter-plugin`, `a/autosave-swap`).
- Commits: `type(track): description` — types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.
- Every PR references an issue (`Closes #N`).
- If your PR modifies a signature already frozen in `CONTRACTS.md`, add the `contract-change` label — it requires approval from every track that consumes that contract (see `CONTRACTS.md` for the list of consumers per contract).

## Definition of Done

See section 6 of `inkling-plan-desarrollo.md` (internal team document).

## Tests

The `parser.ts`/`serializer.ts` modules (round-trip), `autosave.rs`, and `watch.rs` have a mandatory test gate in CI — see section 5 of `inkling-plan-desarrollo.md` for the minimum expected edge cases for each one.