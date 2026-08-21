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
  not `code` — otherwise prose-heavy diffs read as pure code.
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
- **Each cluster carries its reasoning** — one short paragraph on *why* it exists — sitting **above
  the actual hunks** it draws on, in `<details>` (the `html-doc` Visual-evidence rule: the claim in
  prose, the real diff one expand below it, escaped). Those hunks **may overlap** another cluster's
  — two change-types can live on one physical line. Intents partition cleanly; hunks need not.

If the diff has a `specs/` catalog (or the project's documented spec location), read the relevant
spec/concern for the *intent* and link it. If it has none, the review works identically without the
`spec` facet — **never require one.**

## 4. Summarize — a summary that moves with the filter

Lead with a **summary block** that recomputes as facets and change-types are toggled: files
touched, +/− lines, per-change-type line counts. Render the per-change-type line distribution as a
**single-series bar chart** — inline SVG, no chart library. **Load the `dataviz` skill first** for
palette and light/dark-safe form; a review's line distribution *is* a distribution.

## 5. Build the surface (per `html-doc`)

One self-contained, theme-aware file:

- **Two filter axes** — facet and change-type — as combinable checkbox rows with **live counts**.
  The full diff renders filtered by the active selection.
- **Change-type clusters are the primary structure**; the facet filter narrows within and across
  them. Each cluster is its reasoning paragraph + the hunks one expand away.
- Hunks in `<pre>` with add/del styling, every interpolated line **escaped** (`<`, `>`, `&`) so a
  diff can't break the layout or inject markup.
- **Selection state in small vanilla JS.** The document must be **fully readable with JS disabled**
  — clusters, reasoning, and diffs are all present; filtering is the enhancement, never the gate.

## 6. Deliver

Write the `.html` to a sensible path (the user's scratchpad unless they name a location) and hand
it back with the file tools; offer to publish it as an Artifact for a shareable link. Give a
one-or-two-line summary of what the change *is* — don't restate the review in chat; the file is the
deliverable.

## Notes

- **Read-only** is a hard guarantee: this command resolves a diff and writes exactly one output
  file. It never commits, pushes, edits a PR/issue, or changes the branch it reviews.
- Project-agnostic: the tracker, the default branch, the spec location, and any path overrides come
  from the target project's `CLAUDE.md`, never hardcoded here.
- **Known gap — net-diff blindness:** the review reads the net `base...head` diff, so on a
  build-then-revert branch, work the PR body centers on can net to zero and not appear. When the PR
  body names paths that are absent from the net diff, **flag that** rather than silently omit it.
- Complements `report` (facts *about* heterogeneous sources) and `decide` (the forks *between*
  things). A branch review is the third shape: one diff, read and organized. If the change hides a
  decision that still needs making, point at `decide`.
