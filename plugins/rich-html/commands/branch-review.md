---
description: Turn a branch or PR diff into a self-contained, filterable HTML review — the full diff narrowed by facet (code/tests/docs/spec) and regrouped by change-type (why each cluster of changes exists), with a summary that recomputes as you filter. Asks nothing it can infer; reads the diff and its intent, then emits one theme-aware file. Read-only — it never touches the branch it reviews.
argument-hint: <branch | PR#/URL | range> [angle/audience notes]
---

Build a rich HTML **branch review** of **$ARGUMENTS**.

A branch review turns a diff into **one readable, filterable document** — not a wall of hunks. It
reads the change and *why each part of it exists*, then lets a reviewer slice the full diff two
ways: by **facet** (is this code, tests, docs, or spec?) and by **change-type** (what is this
cluster of changes *for*?). It is a document *about* a diff — it **never mutates** what it reviews
(no commits, no pushes, no PR edits, no branch changes). The output is a single self-contained
interactive HTML file.

**Format contract:** the *how* of the file — self-containment / Artifact CSP, theme-awareness,
progressive disclosure, and the **Visual evidence** vocabulary (the diff hunks and the summary
chart are exactly that) — comes from the **`html-doc`** skill. Read
`${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first and conform to it. This command owns only
the *what*: resolving the diff, classifying it, and structuring the review.

## 1. Resolve the diff — pin exactly what is under review

Work out the change set from `$ARGUMENTS`, in priority order:

- **PR number / URL** (`#662`, a GitHub URL) → `gh pr diff <n>` for the diff and `gh pr view <n>`
  for title, body, and threads (the *intent* behind the change). Base = the PR's base branch.
- **A range** (`main...feature`, `a1b2..c3d4`) → use it verbatim.
- **A branch name** → `git merge-base` against the repo's default branch, then diff `base...branch`.
- **Bare / no argument** → the current branch against its merge-base with the default branch.

The diff is the **net** `base...head`. Pin the resolved base and totals in the report header
(`main…feature · 42 files · +1,013 −287`) so the reader knows exactly what they are looking at.
`gh` is only needed for the PR path — a range or branch needs nothing but `git`.

## 2. Classify by facet — the mechanical axis

Tag every change with a **facet**; the filter is `[all] [code] [tests] [docs] [spec]`, combinable:

- **`code`** — non-test source files.
- **`tests`** — test paths and suffixes (`*test*`, `*spec*`, `__tests__`, `*_test.*`, test dirs).
- **`docs`** — comment-only / javadoc-only hunks, `*.md`, doc dirs. **Classify this at the hunk
  level, not the file level:** a code file whose change is a big comment or javadoc block is `docs`,
  not `code` — otherwise prose-heavy diffs read as pure code. A single hunk can **mix facets**: tag
  it by its dominant facet, and where a block inside it is cleanly one facet (a whole javadoc
  paragraph beside code) you may split it into presentational sub-hunks so each carries one tag.
  Don't over-split — this is for legibility, not partition purity.
- **`spec`** — files under a `specs/` catalog (or the project's documented spec location).

This axis is heuristics, not judgment — the lowest-risk part. Let the target project's `CLAUDE.md`
override the path rules if it documents its own layout.

## 3. Cluster by change-type — name each cluster by its intent

This is the axis that makes the review worth more than a diff viewer, and the one that degrades
into a generic dump if you get it wrong. Group the diff by **change-type = why a cluster of
changes exists**, and follow these rules exactly:

- **Name every cluster by what *this* change does** — `jul2026-sync-replay`,
  `java-opts-expansion-fix`, `pipeline-revert` — **never a bare layer word** like `service`,
  `config`, or `migration`. The name is the reasoning in miniature.
- **Treat this vocabulary as a prompt list and an optional tag, not buckets to sort into:**
  `model/repository`, `config`, `service`, `api-surface`, `migration`, `test-scaffolding`,
  `process/convention` (spec records, catalog/changelog, lint config). Use it to *prompt* yourself
  to look, and to *tag* a cluster for filtering — **never make "is this seed or emergent?" a
  question you have to answer.** A cluster can carry a tag *and* be named by its intent.
- **The smell to watch for is generic-layer naming.** If a cluster's *name* is a bare layer word
  instead of a phrase describing what the change does, you have probably missed the point of the
  change — re-read the diff. (Checkable from the names alone, which is the whole point.)
- **All-emergent is fine.** A revert or consolidation has no forward-feature clusters and every
  cluster is change-specific — that is correct, not a gap. Too few tags is never the problem;
  generic names are.
- **The headline change is often the untagged one.** The cluster that is the *actual point* of the
  PR frequently fits no seed word and stays change-specific, while the peripheral clusters (a
  migration, test scaffolding, the spec record) carry seed tags. That asymmetry is expected.
- **Cluster tests by the intent they serve.** When test files back two genuinely different
  production changes, split them into separate clusters named for each; when splitting would force a
  generic name, keep them one. ("Too few tags is never the problem" governs the seed tags — not how
  finely you cluster by intent.)
- **Each cluster carries its reasoning** — one short paragraph on *why* it exists — sitting **above
  the actual hunks** it draws on, in `<details>` (the `html-doc` Visual-evidence rule: the claim in
  prose, the real diff one expand below it, escaped). Those hunks **may overlap** another cluster's
  — two change-types can live on one physical line. Intents partition cleanly; hunks need not.

If the diff has a `specs/` catalog (or the project's documented spec location), read the relevant
spec/concern for the *intent* and link it. If it has none, the review works identically without the
`spec` facet — **never require one.**

## 4. Summarize — a summary that moves with the filter

A compact **summary** beside the filter controls that recomputes as the axes are toggled: files
touched (each distinct file counted once) and ± lines over the **currently visible** (filtered)
selection. Keep it small — a live "showing N files · +A −B" line, not a dashboard.

When a per-change-type **line distribution** genuinely aids scanning (many clusters, lopsided
sizes), add a small **single-series bar chart** — inline SVG, no chart library. **Load the `dataviz`
skill first** for palette and light/dark-safe form, and **pin the axis to the full-diff maximum** so
toggling a filter shortens bars in place without rescaling. It is optional and subordinate: the
review's job is the diff, not the chart, so don't let it crowd the clusters.

## 5. Build the surface (per `html-doc`)

One self-contained, theme-aware file. The canonical shape is a **review tool, not a document dump**
— it should *resemble* a real diff-review UI (GitHub/GitLab: file tree, diff gutters, dense filter
bar) while staying a theme-aware document. The worked example
(`docs/rich-html-branch-review-example.html`) is the reference. Its parts, top to bottom:

- **A "why these change-types" overview** — 1–2 short paragraphs mapping the diff before any detail:
  which cluster is the **headline** (the one carrying behaviour) and why the others exist (the
  support work). It answers "which part actually matters?" from the top, so the reader knows where
  to spend attention.
- **Two combinable filter axes** — change-type × file-type — as chip/checkbox rows with **live
  counts**, sitting at the top where they govern everything below. They **intersect**: a change
  shows only if *both* its change-type and its file-type are selected — and an empty intersection
  (`code` × a docs-only cluster) is a valid, honest result, not a bug.
- **A collapsible file navigator** (left column) — the files grouped by change-type, each jumping to
  its diff; it collapses to a slim reopen tab to give the diffs full width.
- **Change-type clusters as the primary structure** — each leads with its name, tag, line count, and
  **reasoning paragraph**, then renders **its files as inline unified diffs** (add/del gutters, line
  numbers), each file collapsible. Reasoning sits directly above the real diff it explains — not one
  tree-trip away (the `html-doc` Visual-evidence rule).
- Hunks in `<pre>` with add/del styling, every interpolated line **escaped** (`<`, `>`, `&`) so a
  diff can't break the layout or inject markup.
- **Small vanilla JS** for the filters, the tree collapse, and file jumps. The document must be
  **fully readable with JS disabled** — overview, clusters, reasoning, and diffs are all present;
  filtering and navigation are enhancements, never gates.
- **Theme-aware, still a document.** The review-tool look adapts to the reader's light/dark (per
  `html-doc` §2) — it *resembles* a code-review UI but is not a product facsimile, so it respects
  the viewer's theme rather than pinning one. (Pinning a product theme is `mockup`'s posture, not
  this one.)

## 6. Deliver

Write the `.html` to a sensible path (the user's scratchpad unless they name a location) and hand
it back with the file tools; offer to publish it as an Artifact for a shareable link. Give a
one-or-two-line summary of what the change *is* — don't restate the review in chat; the file is the
deliverable.

## Notes

- **Read-only** is a hard guarantee: this command resolves a diff and writes exactly one output
  file. It never commits, pushes, edits a PR/issue, or changes the branch it reviews.
- **Out of scope — annotate-and-emit.** Letting the reader *comment on a change-type* and emit an
  agent-ready prompt from those notes (a `decide`-style payload) is deliberately **not** part of
  branch-review — that write-path is `decide`'s job and would break the read-only guarantee above.
  branch-review stays a read-only viewer; feed its findings to `decide` when you want to act on them.
- Project-agnostic: the tracker, the default branch, the spec location, and any path overrides come
  from the target project's `CLAUDE.md`, never hardcoded here.
- **Known gap — net-diff blindness:** the review reads the net `base...head` diff, so on a
  build-then-revert branch, work the PR body centers on can net to zero and not appear. When the PR
  body describes a path as *changed by this PR* but that path is absent from the net diff, **flag
  it**. A path the body merely *references* for context — an unchanged file, a line number in
  existing code — is not a gap; don't flag those.
- **The two axes are lenses, not a partition.** Facet and change-type can overlap — spec content
  shows up under the `spec` facet *and* a `process/convention` change-type, and on a small PR the two
  axes can nearly coincide. That's fine: each axis is a way to slice the same diff, not a claim that
  they carve disjoint sets.
- Complements `report` (facts *about* heterogeneous sources) and `decide` (the forks *between*
  things). A branch review is the third shape: one diff, read and organized. If the change hides a
  decision that still needs making, point at `decide`.
