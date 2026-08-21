# rich-html — turn your work into interactive HTML deliverables

A short guide to the `rich-html` plugin and its three commands, `report`, `decide`, and
`branch-review`. Each takes the scattered inputs of your work and turns them into a single
**self-contained, interactive HTML file** — to *read*, to *act on*, or to *review*.

> 📖 **Prefer the visual version?** Open [`rich-html-guide.html`](./rich-html-guide.html) in a
> browser for the same guide with the two-track diagram and nicer formatting. This Markdown page is
> the quick, GitHub-readable companion.

```
sources    → report        → a readable document
decisions  → decide        → an expandable surface + one prompt
a diff     → branch-review → a filterable review, grouped by change-type
```

## Install

```
# 1 · register the marketplace (once)
/plugin marketplace add moacyrricardo/skills

# 2 · install
/plugin install rich-html@moacyr-skills

# 3 · activate in the current session
/reload-plugins
```

Once installed the commands are namespaced: `/rich-html:report`, `/rich-html:decide`,
`/rich-html:branch-review`.

## Three tools, one system

Every command produces a **document about your work** — not the product itself (that's what the
`prototype` plugin's `mockup` is for). They share a format contract, the `html-doc` skill, so the
outputs feel like one system.

| | `report` | `decide` | `branch-review` |
| :-- | :------- | :------- | :-------------- |
| **Turns** | heterogeneous sources | pending decisions | a branch/PR diff |
| **Into** | a readable document | an interactive decision surface | a filterable diff review |
| **You** | read it | choose per decision, then run the emitted prompt | filter & read |
| **For** | facts *about* things | the forks *between* things | what a change *does*, and why |

## The shared format — the `html-doc` skill

`report`, `decide`, and `branch-review` all build on `html-doc`, which encodes the non-obvious
rules that make one HTML file work everywhere, stay readable, and carry evidence in the form that
reads fastest:

- **Self-contained** — inline CSS/JS, no network, assets as `data:` URIs. Renders identically as a
  local file, an email attachment, or a Claude Artifact (which enforces a strict CSP).
- **Theme-aware** — respects the reader's light/dark preference with a manual toggle. (A *document*
  adapts to the reader — the opposite of a `mockup`, which matches the product's real theme.)
- **Progressive disclosure** — the conclusion is visible; the evidence is one expand away, in place,
  via native `<details>`. Dense, but never a wall.
- **Visual evidence** — evidence is always the prose claim *plus*, when it helps, a supporting
  artifact under it (never the artifact alone): a code snippet, an inline-SVG flow diagram, a chart
  (via the `dataviz` skill), an existing screenshot, or a schematic divs+CSS layout sketch.

## `report` — sources → a readable document

Read-only. It never changes the things it reports on.

1. **Frame first** — it settles the *audience* and the *angle* (the one question the report is for)
   before gathering, so the result isn't an undifferentiated data dump.
2. **Gather & cross-check** — pulls from whatever sources are present (repo & git, PRs/GitHub, the
   issue tracker, docs & specs, logs) and **cross-checks** them. When two sources disagree, that
   discrepancy is itself a finding — surfaced, not smoothed over.
3. **Conclusion first** — leads with the answer; sections follow; every claim that rests on a source
   carries its proof one expand away, attributed so it's checkable.

```
/rich-html:report the state of the payments migration, for the team
```

## `decide` — decisions → a surface that emits one prompt

`decide` is *not* "ask the user a question in chat." It gathers the decisions **scattered across
your work** into one place, each with the real evidence to choose against, and emits a prompt an
agent can execute.

1. **Collect** — sweeps for genuine pending choices: a concern's `## Options`, unresolved PR-review
   forks, "needs decision" issues, autopilot escalations, `TODO(decide:)` markers.
2. **Model each** — a one-line question, 2–4 concrete options (with a *recommended* one where the
   evidence supports it), and **expandable context** wired to the actual source.
3. **Emit one prompt** — you choose per decision (or "accept all recommended"), and a **Copy prompt**
   button assembles a single, self-standing prompt — one line per decision with its choice and source
   — to hand straight back to an agent.

```
/rich-html:decide everything waiting on me across the open concerns and PRs
```

Guardrail: `decide` always renders the rich surface — it **never** degrades into chat Q&A. The
expandable-context format *is* the value.

## `branch-review` — a diff → a filterable review

Read-only. It never touches the branch it reviews.

1. **Resolve the diff** — from a PR number/URL, a `base...head` range, a branch name, or bare (the
   current branch vs. its merge-base). The exact base and totals are pinned in the header, so you
   know precisely what is under review.
2. **Two filter axes** — every change is tagged by **facet** (`code / tests / docs / spec`) and
   clustered by **change-type**: *why* each cluster of changes exists, **named by its intent**
   (`jul2026-sync-replay`, not a bare `service`). The full diff slices both ways, and a summary —
   files, ± lines, a `dataviz` bar chart of the line distribution — recomputes as you filter.
3. **Reasoning over hunks** — each cluster leads with one paragraph on *why* it exists; the real
   diff is one expand below it.

```
/rich-html:branch-review #662
```

> 📄 **Worked example:** [`rich-html-branch-review-example.html`](./rich-html-branch-review-example.html)
> is this plugin's own PR #20 reviewed *by* `branch-review` — four change-type clusters (the headline
> `branch-review-command` left untagged/change-specific, the rest tagged `api-surface` / `docs` /
> `process/convention`), filterable by facet and change-type with a live-recomputing summary and bar
> chart. It also shows the *axes-are-lenses* caveat in the wild: on this tidy PR the two axes coincide.

## When to use which

- Reaching for the **state of something**, a synthesis, a briefing → **`report`**.
- Facing a pile of **unmade choices** and want to clear them in one pass → **`decide`**.
- Wanting to **understand or review a diff** — what a branch or PR actually changed, and why →
  **`branch-review`**.

If `report` runs into a decision that needs making, it points at `decide`; if `decide` finds
reportable state rather than a fork, it points back at `report`; and if a change hides an unmade
decision, `branch-review` points at `decide` too.

## Changelog

- **2026-08-21 · `branch-review` — a third command (plugin `0.3.0`).** Turns a branch or PR diff
  into a filterable HTML review: the full diff sliced by **facet** (`code / tests / docs / spec`)
  and regrouped by **change-type**, each cluster **named by its intent** rather than a generic
  layer. The change-type rule was hardened by an adversarial redteam across three real PRs — the
  original seed-vs-emergent taxonomy was undecidable when a change fit both, so it became
  name-by-intent with *generic-layer naming* as the checkable smell. Built from specs
  [`001`](../specs/001-concern-branch-review.md) (concern) and
  [`002`](../specs/002-done-branch-review.md) (spec), and dogfooded on its own PR — see the
  [worked example](./rich-html-branch-review-example.html).
- **2026-08-20 · Visual evidence — a fourth `html-doc` rule.** Evidence can now be carried in the
  form that reads fastest — a code snippet, an inline-SVG flow diagram, a chart (via the `dataviz`
  skill), an existing screenshot, or a schematic divs+CSS layout sketch — always *added under* the
  prose claim, never replacing it. Backed by a side-by-side trial:
  [`rich-html-visual-evidence-ab.html`](./rich-html-visual-evidence-ab.html) renders four scenarios
  (code / layout / chart / flow) twice from identical content — prose-only vs. prose + a supporting
  artifact — with a Baseline / Side-by-side / Improved toggle and a scoring rubric.

---

Part of [moacyrricardo/skills](https://github.com/moacyrricardo/skills) · see the
[README](../README.md) for the full plugin reference.
