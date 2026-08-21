# 001 — branch-review: a filterable diff "review" for rich-html

## Problem

`/rich-html:report`, run by hand against PR #662 (a ~1k-line change in the boletim repo),
produced something better than a report: a **diff review**. One self-contained HTML file
(~157 KB) with

- a **facet filter** — `[all] [code] [javadoc/comments] [tests] [spec]`, combinable — that
  narrows the full diff to the kind of change you want to look at;
- a **change-type dimension** — the diff regrouped by *why a cluster of changes exists*
  (in #662 the buckets that actually emerged were `service`, `repository`, `config`,
  `migration`, `modeling`, plus one custom to this change: `extractor-rewiring`), each carrying
  the reasoning of the implementation, not just the hunks;
- a **summary by filter** — line-change counts and the like, recomputed as you filter.

> **Seed artifact (evidence, not checked in):** the report that started this lives at
> `file:///mnt/mydata/repos/boletim/spec-090-pr662-review.html`. It is *not* copied into
> `001-assets/` on purpose — it embeds boletim's private diff and this repo is the public
> marketplace. Regenerate it to inspect; don't vendor it here.

The tension worth exploring: this was valuable enough to want on demand, but it arrived as an
*ad-hoc report*. The question is what, if anything, to build so it is reproducible — and how to
encode the one part that made it good (the change-type synthesis with a custom bucket) so it
doesn't degrade into a generic diff dump.

## Hypotheses / Options

### A. A new command — `/rich-html:branch-review`
A third command beside `report` and `decide`, with a fixed content model: gather a branch/PR
diff → classify by facet → cluster by change-type → synthesize reasoning per cluster → emit one
filterable HTML file.

- **Pro:** codifies the structure so nobody re-derives it each run; a named command is
  discoverable; the facet+change-type+summary model is stable enough (proven once by #662) to be
  worth freezing.
- **Con:** command proliferation; heavy overlap with `report` (a review *is* a report about a
  diff); risks two things to maintain that share 80% of their guidance.

### B. A documented "review" recipe/mode of `report`
Keep it as `report`, but add a review-shaped section to `report.md` (and possibly `html-doc`)
that recognizes "the subject is a branch/PR diff" and reaches for the facet + change-type
structure.

- **Pro:** no new surface; leans on `report`'s existing gather/cross-check/structure machinery;
  one place to maintain.
- **Con:** less discoverable (nobody types `/rich-html:report` expecting a diff viewer); the
  filterable-diff interaction model is enough of its own thing that burying it as a report mode
  may undersell it.

### C. Do nothing — it's already achievable ad-hoc
#662 proves `report` can already do this with a good enough prompt.

- **Pro:** zero build.
- **Con:** the result depended on a carefully-worded one-off prompt; not reproducible; the
  change-type/custom-bucket insight isn't captured anywhere.

**Current lean:** A or B over C — the value is real and worth capturing. The A-vs-B fork is the
load-bearing decision this concern exists to tee up (see Open Questions).

## Design tensions any resolution must handle

1. **The change-type taxonomy is seed + emergent — and the emergent half is the whole point.**
   There is a reusable seed of buckets (`model/repository`, `config`, `service`, `api-surface`,
   `migration`) *and* every change has at least one bucket specific to it (`extractor-rewiring`
   for #662). The skill must **actively incentivize naming the change-specific bucket** rather
   than forcing everything into the seed. This is the defining problem: the whole thing is only
   as good as its resistance to a lazy "everything is `service`" bucketing. A fixed enum fails;
   a free-for-all loses cross-review consistency. The answer is probably "seed as a starting
   vocabulary, explicit instruction to add + prefer change-specific buckets when the diff
   warrants."

2. **Facet filtering is mechanical and low-risk.** `code / tests / javadoc-comments / spec`
   classification is path + content heuristics. Park it; it is not where the design effort goes.

3. **The reasoning-per-change-type synthesis is where this hallucinates.** Grouping a diff by
   *why each cluster exists* is synthesis, not diff mechanics — high value, high risk. This is
   the redteam target once a design exists.

4. **Specs are supporting context, never required.** A branch-review reads a `specs/` catalog
   *when present* to explain intent, but must not depend on one — that protects rich-html's
   stated "no spec or tracker coupling" positioning (see `plugin.json`). #662 happened to have
   spec 090; a config-bump PR with no spec must review just as well.

5. **Scale.** A 1k-line diff fits one file comfortably (#662 was 157 KB, far under the 16 MB
   Artifact ceiling). Progressive disclosure (`html-doc` §3) handles density; larger diffs need
   a stance on truncation/summarization, logged not silent (`html-doc` accessibility rules).

## Open Questions

Must be settled before this becomes a spec:

1. **Command vs. recipe (A vs. B)?** This gates everything else — a new `branch-review.md` +
   plugin surface, versus edits to `report.md`/`html-doc`. *Owner: user.*
2. **How does the skill encode the seed-plus-emergent taxonomy** so it reliably produces the
   custom bucket instead of collapsing into the seed? Needs an adversarial pass (tension 1/3).
3. **Redteam the change-type synthesis across varied diffs** — a migration-heavy PR, a pure
   refactor, a config bump — to see whether the taxonomy holds or degrades. This is the one
   place a fable-5 fan-out earns its keep (breadth across real diffs).
4. **Diff source contract** — branch-vs-base range resolution, PR (`gh`) vs. local branch,
   how the base is chosen. Mechanical but must be pinned in the spec.
5. **Facet classification heuristics** — path/content rules for `code / tests / docs / spec`,
   and how "spec" is even detected in a project that may not use a `specs/` catalog.

## Next step

Resolve Q1 (A vs. B), then run the Q2/Q3 redteam against a proposed default design. If it
survives, promote to a spec (`002-todo-...`), which resolves this concern with a WARNING block.
