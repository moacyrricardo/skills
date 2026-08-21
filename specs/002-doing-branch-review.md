Status: doing
Branch: moacyrricardo/spec-002-branch-review

# 002 — branch-review: a filterable diff review command for rich-html

## Context

Running `/rich-html:report` by hand against boletim PR #662 (~1k lines) produced something better
than a report: a **filterable diff review** — full diff, narrowed by a facet filter
(`code / tests / docs / spec`), regrouped by *why each cluster of changes exists*, with a summary
that recomputes as you filter. It was valuable enough to want on demand, but it arrived as a
one-off prompt and wasn't reproducible.

Concern **001** explored this and settled the load-bearing fork: **Q1 → a new thin command**
(not a `report` recipe), because a diff review is a *different artifact* from a report (a
filterable diff viewer organized by change-type, not conclusion-first prose) and the shared file
HOW already lives in `html-doc`, so a new command duplicates nothing. The change-type taxonomy —
the part that made #662 good and the part most likely to degrade into a generic diff dump — was
hardened by an adversarial fable-5 redteam across three real PRs (#640 migration, #657 config,
#622 revert). See `001-assets/design-proposal.md` for the full pre-spec design and the redteam
findings; this spec is the decided, buildable form.

## Decision

Add a third rich-html command, **`/rich-html:branch-review`**, alongside `report` and `decide`:

- **Thin, like its siblings.** It owns only the diff-review *content model*; all file HOW
  (self-containment, theme, progressive disclosure, visual evidence) comes from **`html-doc`**, and
  the summary chart composes **`dataviz`**. It adds no new format rules.
- **Read-only**, the same hard guarantee as `report`: it inspects a diff and writes exactly one
  HTML file — never commits, pushes, edits an issue, or mutates the branch.
- **Two filter axes** over the full diff: a **facet** dimension (mechanical, path/hunk-based) and a
  **change-type** dimension (synthesis: *why* each cluster exists).
- **Change-type is named by intent, not sorted into a fixed taxonomy** (the redteam killed the
  seed-vs-emergent binary — see Implementation §4). This is the command's defining rule.
- **Specs are supporting context, never required** — protects rich-html's project-agnostic stance.

## Implementation

### 1. New command file `plugins/rich-html/commands/branch-review.md`

Front matter mirrors the siblings:

```
---
description: Turn a branch or PR diff into a self-contained, filterable HTML review — the full
  diff narrowed by facet (code/tests/docs/spec) and regrouped by change-type (why each cluster of
  changes exists), with a summary that recomputes as you filter. Read-only; never mutates the branch.
argument-hint: <branch | PR#/URL | range> [angle/audience notes]
---
```

Body structure (prose sections, like `report.md`/`decide.md`): **Format contract** pointing at
`${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first; then the numbered steps below. It owns the
*what*; `html-doc` owns the *how*.

### 2. Diff source contract

Resolve the change set from `$ARGUMENTS`, in priority order:

- **PR number / URL** (`#662`, a GitHub URL) → `gh pr diff <n>` + `gh pr view <n>`
  (title/body/threads for intent). Base = the PR's base branch.
- **A range** (`main...feature`, `a..b`) → use verbatim.
- **A branch name** → `git merge-base` vs. the repo's default branch, diff `base...branch`.
- **Bare / no arg** → current branch vs. its merge-base with the default branch.

Diff is `base...head` (net). Pin the resolved base + totals in the report header
(`main…feature · 42 files · +1,013 −287`). `gh` is optional — range/branch paths need only `git`.

### 3. Facet dimension (mechanical, low-risk)

Tag each change by facet; the filter is `[all] [code] [tests] [docs] [spec]`, **combinable**:

| facet | heuristic |
|-------|-----------|
| `code` | non-test source files |
| `tests` | test paths/suffixes (`*test*`, `*spec*`, `__tests__`, `*_test.*`, test dirs) |
| `docs` | comment-only / javadoc-only hunks, `*.md`, doc dirs — **hunk-level, not file-level** |
| `spec` | files under a `specs/` catalog (or the project's documented spec location) |

**Docs is hunk-level** (redteam ③): a code file whose change is a large comment/javadoc block
counts `docs`, not `code`. The target project's `CLAUDE.md` may override the path rules.

### 4. Change-type dimension — name by intent (the defining rule)

The diff is regrouped by **change-type = why a cluster of changes exists**. The redteam replaced
the original seed-vs-emergent binary (undecidable when a change fit both, which made an "all-seed"
smell circular). The rule the command must encode:

- **Name every cluster by its intent — what *this* change does** (`jul2026-sync-replay`,
  `java-opts-expansion-fix`, `pipeline-revert`) — **never a bare layer word** (`service`, `config`,
  `migration`). The name is the reasoning in miniature.
- **The seed vocabulary is a prompt list + an optional filter tag, not buckets to sort into.**
  `model/repository`, `config`, `service`, `api-surface`, `migration`, `test-scaffolding`,
  `process/convention` (spec-record, catalog/changelog, lint config) prompt the reviewer to look,
  and tag a cluster for filtering. **"Seed or emergent?" is never a question to adjudicate** — a
  cluster can carry a seed tag *and* be named by intent.
- **The smell is generic-layer naming, not "all-seed."** A cluster *named* by a bare layer word is
  the signal the reviewer missed the point — checkable from the names alone, no need to already
  know the point.
- **All-emergent is normal** (a revert/consolidation has no forward-feature clusters); too-few-seed
  is never the smell. This closes the loophole of laundering a lazy review with seed-word names.
- **Each cluster carries its reasoning** — one short paragraph on *why* — above the hunks it draws
  on, in `<details>` (html-doc Visual-evidence: claim + real diff under it). Hunks **may overlap**
  another cluster's (#657: one `JAVA_OPTS` line is both the fix and the retune). Intents partition;
  hunks need not.

### 5. Summary by filter

A summary block that **recomputes as facets/change-types are filtered**: files touched, +/− lines,
per-change-type line counts. Render the per-change-type distribution as a single-series bar chart
via **`dataviz`** (load it first) — a review's line distribution *is* a distribution.

### 6. HTML interaction model (via `html-doc`)

One self-contained, theme-aware file. Facet + change-type are the two filter axes (checkbox rows,
live counts). Full diff renders filtered by the active selection; hunks in `<pre>` with add/del
styling, **escaped**. Change-type clusters are the primary structure; the facet filter narrows
within/across them. Selection state is small vanilla JS. **Fully readable with JS off** (clusters,
reasoning, and diffs all present; filtering is the enhancement).

### 7. Specs are context, never required

If a `specs/` catalog (or the project's documented spec location) exists, read the relevant
spec/concern for *intent* and link it. If none, the review works identically minus the `spec`
facet. No coupling.

### 8. Plugin + docs

- **`plugins/rich-html/.claude-plugin/plugin.json`**: bump `version` `0.2.0 → 0.3.0` (new command
  = minor under 0.x) and extend `description` to name the third command.
- **`docs/rich-html-guide.md` + `docs/rich-html-guide.html`**: add `branch-review` as a third
  command (the "two tools, one system" framing becomes three), and a Changelog entry.

## Known Gaps

- **Net-diff blindness (redteam ⑤):** the review reads the `base...head` net diff, so on a
  build-then-revert branch (#622) work the PR body centers on can net to zero and vanish. Net diff
  is the right default; the command should **flag** when the PR body names paths absent from the net
  diff, rather than silently omit — deferred as a refinement, not a v1 blocker.
- **Facet precision below the hunk:** a single line that is both code and comment can't be split;
  accepted as good-enough.
- **Not yet dogfooded off boletim:** the redteam used boletim PRs. First non-boletim run may surface
  path-heuristic gaps the target `CLAUDE.md` override is meant to absorb.
