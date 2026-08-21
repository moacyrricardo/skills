# branch-review — proposed default design (pre-redteam)

> Feeds concern **001**. This is the design the Q2/Q3 redteam attacks. If it survives, it is
> promoted to spec **002-todo-branch-review**; this file stays as the concern's asset (the
> pre-redteam baseline).

Resolved going in: **Q1 = A** — a new thin command `/rich-html:branch-review`, owning only the
diff-review *content model* and delegating all file HOW to `html-doc` (like `report`/`decide`).

## Command shape

`plugins/rich-html/commands/branch-review.md`, `argument-hint: <branch/PR/range> [angle notes]`.
Read-only (same hard guarantee as `report`): inspects a diff, writes exactly one HTML file, never
mutates. Composes `html-doc` (format HOW) and `dataviz` (the summary chart). One output file.

## 1. Diff source contract

Resolve the change set, in priority order, from `$ARGUMENTS`:

- **PR number / URL** (`#662`, a GitHub URL) → `gh pr diff <n>` + `gh pr view <n>` for
  title/body/threads. Base = the PR's base branch.
- **A range** (`main...feature`, `abc123..def456`) → use verbatim.
- **A branch name** → `git merge-base` against the repo's default branch, diff `base...branch`.
- **Bare / no arg** → current branch vs. its merge-base with the default branch.

Pin the base explicitly in the report header ("`main`…`feature` · 42 files · +1,013 −287") so the
reader knows exactly what is under review. `gh` optional — a range/branch path needs only `git`.

## 2. Facet dimension — mechanical classification (low-risk)

Every file (and where cheap, every hunk) is tagged with one **facet** by path + content:

| facet | heuristic |
|-------|-----------|
| `code` | source files that aren't tests (language src dirs, non-`*test*` paths) |
| `tests` | test paths/suffixes (`*test*`, `*spec*` in a test dir, `__tests__`, `*_test.go`, …) |
| `docs` | `javadoc/comments`-only hunks, `*.md`, doc dirs — **hunk-level, not file-level** |
| `spec` | files under a `specs/` catalog (or the project's documented spec location) |

The facet filter is `[all] [code] [tests] [docs] [spec]`, **combinable** (multi-select). This is
heuristic-driven and the lowest-risk part; the spec pins the exact rules and lets the target
project's `CLAUDE.md` override paths.

**Docs is hunk-level, deliberately (redteam ③).** A code file whose change is a large comment /
javadoc block must count as `docs`, not `code` — #640 buried ~45 lines of javadoc inside two code
files, and a file-level facet scored `docs`=0 on a diff where prose was half the content. Classify
comment-only hunks as `docs` even inside a `code` file.

## 3. Change-type dimension — name by intent (THE hard part, Q2 — post-redteam)

The diff is grouped by **change-type**: *why a cluster of changes exists*. This is the part that
made #662 good and the part that can degrade into a generic diff dump. The redteam (see the
findings section below) rewrote this rule — the original "seed vs. emergent bucket" binary was
**undecidable** when a change fit both (a Flyway file is a seed `migration` *and* has a specific
why), which made the old "all-seed is a smell" check circular. The corrected encoding:

- **Every cluster is named by its intent — what *this* change does — never a bare layer word.**
  `jul2026-sync-replay`, `java-opts-expansion-fix`, `pipeline-revert` — not `service`, `config`,
  `migration`. The name is the reasoning in miniature.
- **The seed vocabulary is a prompt list and an optional filter tag, not a set of buckets to sort
  into.** `model/repository`, `config`, `service`, `api-surface`, `migration`, `test-scaffolding`,
  and `process/convention` (spec-record, catalog/changelog, lint config) are there to *prompt* the
  reviewer to look, and to *tag* a cluster for filtering — **"is this seed or emergent?" is never a
  question the reviewer has to adjudicate.** A cluster can carry a seed tag and still be named by
  its intent.
- **The smell is generic-layer naming, not "all-seed."** If a cluster's *name* is a bare layer
  word rather than a phrase describing what the change does to that layer, that is the signal the
  reviewer missed the point — and it is checkable **from the names alone**, without already knowing
  the point (which is what made the old rule circular).
- **All-emergent is normal and fine.** A revert or consolidation (#622) has *no* forward-feature
  clusters — the seed tags legitimately score zero. Too-few-seed is never the smell; generic naming
  is. This closes the gaming loophole (you can't launder a lazy review by inventing seed-word names,
  because seed-word names *are* the smell).
- **Each cluster carries its reasoning** — one short paragraph on *why* it exists — sitting **above
  the hunks it draws on** (visual-evidence rule: claim + the real diff under it, in `<details>`),
  so the synthesis is checkable, not asserted. Those hunks **may overlap another cluster's**: two
  change-types can live on one physical line (#657's `JAVA_OPTS` line is both the expansion fix and
  the retuned values). Intents partition; hunks need not — don't force a clean split.

## 4. Summary by filter

A summary block that **recomputes as facets/change-types are filtered**: files touched, +/−
lines, per-change-type line counts. The per-change-type distribution is a single-series bar chart
via **`dataviz`** (load it first) — a review's line-distribution *is* a distribution, the form a
report reaches for most.

## 5. HTML interaction model (via html-doc)

One self-contained, theme-aware file. Facet + change-type are the two filter axes (checkbox rows,
live `.fcount`). The full diff renders filtered by the active selection; each hunk in
`<pre class="diff">` with `.add`/`.del` lines, escaped. Change-type clusters are the primary
structure; the facet filter narrows within/across them. Selection state is small vanilla JS
(`html-doc` §3 reserves JS for exactly this). Fully readable with JS off (clusters + reasoning +
diffs all present; filtering is the enhancement).

## 6. Specs are context, never required (tension 4)

If a `specs/` catalog (or the project's documented spec location) exists, a review reads the
relevant spec/concern to explain *intent* and links it. If none exists, the review works
identically minus the `spec` facet. No coupling — protects rich-html's project-agnostic stance.

## Known gaps (deferred, not blockers)

- **Net-diff blindness (redteam ⑤).** A review reads the `base...head` *net* diff, so on a
  build-then-revert branch (#622: 29 commits) work the PR narrative centers on can net to zero and
  vanish from the review. Net diff is the right default (you review the net change); the spec
  should have the review *flag* when the PR body references paths absent from the net diff, rather
  than silently omit them — but this is a refinement, not a v1 blocker.
- **Facet precision below the hunk.** Even hunk-level facets can't split a single line that is both
  code and comment; accepted as good-enough.

## Redteam findings (Q2/Q3) — what this design survived

Adversarial fable-5 pass (classify → skeptic-refute) across three real boletim PRs of different
shapes, none of them the #662 seed (avoiding circularity): **#640** migration, **#657** config/ops,
**#622** revert/consolidation (the honeypot for the predicted "forced emergent bucket" misfire).

- **Verdict: the taxonomy held.** Overall — #622 *holds*, #640 and #657 *partial*. The predicted
  weakness **did not reproduce**: the emergent-naming rule misfired on none of the three (a revert
  *does* have a nameable point). Every bucket produced was judged *real* or (once) *mislabeled* —
  never invented.
- **① (fixed above, §3)** seed/emergent binary undecidable when both fit → the "all-seed smell" was
  circular. Reframed to name-by-intent + generic-layer-naming as the checkable smell.
- **② (fixed above, §3)** all-emergent is legitimate (reverts) and now stated; gaming loophole
  closed.
- **③ (fixed above, §2)** docs facet made hunk-level.
- **④ (fixed above, §3)** `process/convention` added to the seed/tag vocabulary for spec-record /
  catalog / changelog clusters.
- **⑤ (deferred, Known gaps)** net-diff blindness on build-then-revert branches.

Raw run: workflow `branch-review-taxonomy-redteam` (6 agents, 0 errors), journal at the session's
`subagents/workflows/…/journal.jsonl`. The design is judged ready to promote to a spec.
