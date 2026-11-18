# rich-html — turn your work into interactive HTML deliverables

A short guide to the `rich-html` plugin and its four commands, `report`, `decide`,
`branch-review`, and `presentation`. Each takes the scattered inputs of your work and turns them
into a single **self-contained, interactive HTML file** — to *read*, to *act on*, to *review*, or
to *present*.

> 📖 **Prefer the visual version?** Open [`rich-html-guide.html`](./rich-html-guide.html) in a
> browser for the same guide with the two-track diagram and nicer formatting. This Markdown page is
> the quick, GitHub-readable companion.

```
sources    → report        → a readable document
decisions  → decide        → an expandable surface + one prompt
a diff     → branch-review → a filterable review, grouped by change-type
a story    → presentation  → a full-screen slide deck
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
`/rich-html:branch-review`, `/rich-html:presentation`.

## Four tools, one system

Every command produces a **document about your work** — not the product itself (that's what the
`prototype` plugin's `mockup` is for). They share a format contract, the `html-doc` skill, so the
outputs feel like one system.

| | `report` | `decide` | `branch-review` | `presentation` |
| :-- | :------- | :------- | :-------------- | :------------- |
| **Turns** | heterogeneous sources | pending decisions | a branch/PR diff | a story and its sources |
| **Into** | a readable document | an interactive decision surface | a filterable diff review | a full-screen slide deck |
| **You** | read it | choose per decision, then run the emitted prompt | filter & read | present it, slide by slide |
| **For** | facts *about* things | the forks *between* things | what a change *does*, and why | telling a story to an audience |

## The shared format — the `html-doc` skill

`report`, `decide`, `branch-review`, and `presentation` all build on `html-doc`, which encodes the non-obvious
rules that make one HTML file work everywhere, stay readable, and carry evidence in the form that
reads fastest:

- **Self-contained** — inline CSS/JS, no network, assets as `data:` URIs. Renders identically as a
  local file, an email attachment, or a Claude Artifact (which enforces a strict CSP).
- **Theme-aware** — respects the reader's light/dark preference with a manual toggle. (A *document*
  adapts to the reader — the opposite of a `mockup`, which matches the product's real theme.)
- **Progressive disclosure** — the conclusion is visible; the evidence is one expand away, in place,
  via native `<details>`. Dense, but never a wall.
- **Visual evidence** — evidence is always the prose claim *plus*, when it helps, a supporting
  artifact under it (never the artifact alone): a code snippet, an inline-SVG flow or structure
  diagram (a system map, data flow, actor steps, service, sequence, schema or class sketch), a chart
  (via the `dataviz` skill), a table, an existing screenshot, or a schematic divs+CSS UI sketch.

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
   forks, "needs decision" issues, autopilot escalations, `TODO(decide):` / `FIXME` markers.
2. **Model each** — a one-line question, then a **What** paragraph (what is being decided) and a
   **Why** paragraph (why it needs deciding now), and — when something can be drawn — a **today
   visual** of the status quo with the fork marked (UI sketch, schema, use-case flow, service or
   sequence diagram, class/module diagram, timeline). Then 2–4 options, each a **How** paragraph plus
   an **outcome figure** in the same form showing how things look if you choose it; its gains,
   losses and "wrong call when…" sit collapsed and open when you pick it. Every card also has
   **Defer — leave open**. Visuals are nudged, never forced: a naming call or a policy stays prose.
3. **Conflicts** — when options on different cards can't both be done, picking one disables the
   other with the reason shown; un-picking re-enables it. Softer tensions only warn.
4. **Emit one prompt** — you choose per decision (or "accept all recommended"), and the page shows a
   fixed-format prompt — per decision its choice, gains, accepted losses and "revisit if", plus its
   source — with **Copy prompt** to hand it straight back to an agent. Picks survive a reload.

```
/rich-html:decide everything waiting on me across the open concerns and PRs
```

Guardrail: `decide` always renders the rich surface — it **never** degrades into chat Q&A. The
evidence-backed card *is* the value.

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
> is this plugin's own PR #20 reviewed *by* `branch-review` — a review-tool layout with a collapsible
> file navigator, a "why these change-types" overview, and two combinable, **intersecting** filters
> (change type × file type) over four clusters of inline unified diffs (the headline
> `branch-review-command` untagged, the rest tagged `api-surface` / `docs` / `process/convention`).
> It also shows the *axes-are-lenses* caveat in the wild: on this tidy PR the two axes coincide.

## `presentation` — a story → a full-screen slide deck

Read-only. It never changes its sources and never re-captures the product.

1. **Frame** — your story as **beats**, in order, and the effect each should have ("it feels
   bureaucratic" → the numbered steps it takes today, with a tally); audience and occasion.
2. **Gather** — real sources only: transcripts, specs, PRs, existing screenshots and GIFs, the code
   for exact UI copy. Every image is looked at before it's captioned; quotes are verbatim with a
   timestamp or line.
3. **Shape** — one claim per slide, as its headline; one figure that proves it, picked by what the
   beat is about (a quote, an annotated screenshot, a step path, a UI miniature, before/after, a
   GIF, a chart); a source line at the foot.
4. **Build and check** — a 16:9 stage that always fits the window; ←/→, Space, swipe, **F** for full
   screen, **N** for speaker notes; a print view and a no-JS fallback (slides stack). Then every
   slide is rendered headless and looked at — overflow, misplaced callouts, captions that overstate
   their asset — and fixed.

```
/rich-html:presentation the onboarding drop-off: what users told us, the 7-step signup today, the
2-step fix, the prototype GIF — for Friday's product review
```

The hand-back ends with **check before you present**: slides whose asset shows a neighbouring
case, example data, and facts it couldn't verify.

## When to use which

- Reaching for the **state of something**, a synthesis, a briefing → **`report`**.
- Facing a pile of **unmade choices** and want to clear them in one pass → **`decide`**.
- Wanting to **understand or review a diff** — what a branch or PR actually changed, and why →
  **`branch-review`**.
- **Telling a story** to an audience — a problem, how it is today, the fix, the proof →
  **`presentation`**.

If `report` runs into a decision that needs making, it points at `decide`; if `decide` finds
reportable state rather than a fork, it points back at `report`; and if a change hides an unmade
decision, `branch-review` points at `decide` too.

## Changelog

- **2026-11-14 · `presentation`: a slide-deck command (plugin `0.5.0`).** A story's beats and
  their real sources become a self-contained, full-screen 16:9 deck — one claim per slide with the
  figure that proves it, keyboard/button/swipe navigation, speaker notes, print and no-JS views —
  checked by rendering every slide. Distilled from an audit of a hand-prompted deck; see spec
  [`003`](../specs/003-done-rich-html-presentation.md). `html-doc`'s theme toggle no longer
  ignores its first click when the OS is dark.
- **2026-11-14 · `decide` cards rebuilt around what / why / how (plugin `0.4.0`).** Each decision
  carries a visible **What** and **Why** paragraph and, when the fork is drawable, a today visual
  chosen by subject; each option gets a **How** paragraph and an outcome figure in the same form,
  with its gains, losses and "wrong call when…" collapsed until picked. Options that contradict
  across cards disable each other with the reason shown; every card can be deferred; the prompt has
  a fixed format and picks persist. `html-doc` gains state colors, schema / class / sequence
  diagrams, tables, an optional step-through for timing diagrams, and the same-baseline rule for
  compared figures. Visuals are nudged, not forced. Distilled from hand-prompted runs that
  outperformed the default shape.
- **2026-08-24 · `branch-review` output redesigned into a review tool (plugin `0.3.1`).** The
  generated file now *resembles a real diff-review UI* — a collapsible file navigator, a "why these
  change-types" overview, and two combinable **intersecting** filters (change type × file type) over
  change-type clusters of inline unified diffs — while staying a theme-aware document. The command's
  §5 now specifies this shape. (Annotating a change-type and emitting a `decide`-style prompt was
  explored but **deliberately left out** — that write-path belongs to `decide`; `branch-review`
  stays read-only.)
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
