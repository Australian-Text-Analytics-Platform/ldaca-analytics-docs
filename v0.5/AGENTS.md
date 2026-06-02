# AGENTS.md — ldaca-wordflow-docs

Conventions for editing this docs repo. Architecture and layout live in
[`README.md`](README.md); this file covers the operational decision an editor
or agent must make before pushing. **Read the "Delivery decision" section before
touching `registry.json`'s `meta.version`.**

## Delivery decision: when to bump `meta.version` (in `registry.json`)

`meta.version` is the **delivery trigger** for already-installed desktop apps —
**not** a per-commit version. The desktop app's backend (`core/docs_sync.py` in
the app repo) mirrors the published docs into a local cache
(`~/.cache/ldaca_wordflow/docs/content`) and **only re-syncs when the remote
`meta.version` differs from the client's cached marker**. Bumping it makes every
installed `0.5.x` client re-download the whole docs set on its next launch.

**So bumping `meta.version` is a deliberate decision, never automatic.** It is
heavier than a commit, lighter than an app release tag — think of it as a
"publish / deliver to users" choice.

- **Do NOT bump** for routine commits — typo fixes, small wording tweaks, a
  single new anchor, formatting. Commit and push them WITHOUT a bump. They still
  deploy to GitHub Pages, so the **web app and fresh installs get them
  immediately**; existing desktop caches simply pick them up at the next real
  bump. Churning every user with a re-download for a comma is not worth it.
- **DO bump** once the change is worth delivering: a meaningful **batch** of
  edits has accumulated, OR a single change is **important enough** to push to
  all installed clients now (e.g. corrected citation / licensing / legal text, a
  materially wrong instruction, docs for a new feature).
- When unsure whether a change clears the bar, **ask the maintainer** rather than
  bumping by reflex.

> Note: committing maintainer-only files (like this `AGENTS.md`) or other
> non-user-facing changes never warrants a bump.

### How to bump (once you've decided to)

Do it on **both** `v0.5` and `main` so the active line and the editing branch agree:

1. Make the markdown edits (see README → "Editing").
2. In `registry.json`: set `meta.version` `0.5.N → 0.5.N+1` and refresh
   `meta.generated` to today's date.
3. Commit + push both branches → `publish.yml` redeploys GitHub Pages.
4. Refresh the app's bundled offline floor: run
   `frontend/scripts/sync-bundled-docs.mjs` in the app repo (or let the desktop
   bundle build do it).
5. In the master repo, bump the `ldaca_wordflow_docs` submodule pointer.

### Versioning scheme for `meta.version`

`meta.version = 0.5.<docrev>` — an **independent, monotonic docs-content
counter** scoped to the `v0.5` docs line. It is deliberately **decoupled from the
app version** (the `v0.5` docs branch serves *all* `0.5.x` apps) and is **never
semver-parsed**. Its only roles are (a) the opaque change-token compared by
`docs_sync.py` and `sync-bundled-docs.mjs`, and (b) the display label in the
app's `DocsEolBanner.tsx`. So just increment the last segment on each delivery —
reaching `0.5.40` after 40 deliveries is perfectly fine. Do **not** reset or
realign it when the app version bumps.

(The separate `meta.eolDate` field is what actually drives the deprecation
banner's show/hide — see README → "EOL".)
