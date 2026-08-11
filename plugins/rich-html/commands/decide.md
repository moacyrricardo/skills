---
description: Aggregate the decisions currently waiting on you — from concerns, PR review threads, issue questions, TODOs, autopilot escalations — into one rich, interactive HTML surface where each decision expands in place to its full context, you pick an option per decision, and the page emits a single prompt to hand back to an agent to execute them all. Not chat Q&A — a durable surface where a decision is made against real evidence.
argument-hint: <where to look for pending decisions> [scope notes]
---

Build an interactive **decision surface** for the decisions pending in **$ARGUMENTS**.

`decide` is not "ask the user a question." It gathers the **open decisions scattered across your
work** into one place, presents each with the *real evidence* needed to choose, records your
choices, and emits **one prompt** that an agent can run to carry them out. The rich, expandable
format **is** the value — it is what separates deciding-against-evidence from an LLM interrogating
you line by line in chat.

**Format contract:** the *how* of the file — self-containment / Artifact CSP, theme-awareness, and
the progressive-disclosure (expandable-context) pattern — comes from the **`html-doc`** skill. Read
`${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first and conform to it. This command owns the
decision model and the prompt it emits.

## 1. Collect the pending decisions

Sweep the sources in scope for things genuinely **waiting on a human choice** — not facts, not
work-in-progress, but forks where someone must pick:

- **Concerns / design docs** — a `specs/` concern's `## Options` / `## Hypotheses`, or a
  `#comment:` asking for a call.
- **PR review threads** — unresolved questions, "should we A or B?", requested-changes forks.
- **Issue tracker** — issues flagged "Needs decision", open questions on a thread (via the
  project's tracker CLI if `CLAUDE.md` documents one).
- **Autopilot / automation escalations** — blockers a background agent parked for a human (draft-PR
  "🚧 Blocked — needs decision" sections, escalation comments).
- **In-code** — `TODO(decide):` / `FIXME` markers that encode an unmade choice.

Skip anything that is a task rather than a decision. If nothing genuine is pending, say so plainly
rather than manufacturing choices.

## 2. Model each decision

For every decision, assemble:

- **A one-line question** — the fork, stated so it can be answered by picking.
- **Options** — 2–4 concrete choices, each with a short consequence. Mark a **recommended** option
  when the evidence favors one (with a one-line why); never invent a recommendation you can't
  support.
- **Expandable context** — the real evidence behind the fork, shown in place via `<details>`: the
  concern's options section, the PR's changed files / the specific thread, the issue excerpt, the
  code around the marker. This is the point of the surface — wire it to the *actual* source, not a
  paraphrase. Escape interpolated text.
- **Source link** — where it came from, so a choice is traceable.

## 3. Build the surface (per `html-doc`)

One self-contained interactive HTML file:

- Each decision is a **card**: question + options as selectable controls (radio-style), collapsed
  by default, expanding in place to its context.
- **Selection state in vanilla JS** — clicking an option records it; a per-decision "chosen" state
  drives the emitted prompt.
- Convenience actions: **"Accept all recommended"** (one click pre-selects every recommended
  option) and **"Reset"**.
- **Guardrail:** the surface always renders as a rich document. It must **never** degrade into a
  chat Q&A fallback — the format is the deliverable, even for a single decision.

## 4. Emit one prompt — the payload

The surface's output is a **single prompt string** assembled from the selections, revealed by a
**"Copy prompt"** button (enabled once every decision has a choice). The prompt is written to be
handed straight back to an agent and must be **self-standing**:

- Lead with a one-line instruction ("Execute the following decisions in <repo>:").
- One numbered line per decision: the decision, **the chosen option**, and its source reference
  (PR#, issue id, concern number) so the agent can act without re-deriving context.
- Plain text an agent can act on directly — this payload is *separate* from the HTML, never
  embedded chrome inside the document.

## 5. Deliver

Write the `.html` to a sensible path (scratchpad unless the user names one) and hand it back with
the file tools; offer to publish as an Artifact for a shareable link. Tell the user the flow in one
line: open it → expand context → choose → **Copy prompt** → paste back here (or into a fresh
session) to execute.

## Notes

- **Building the surface is read-only** — collecting decisions and rendering the file changes
  nothing. Execution happens only when the user runs the emitted prompt, deliberately, later.
- Project-agnostic: tracker/build specifics come from the target project's `CLAUDE.md`.
- Complements `report` (facts *about* things) — `decide` is for the forks *between* things. If a
  source turns out to hold reportable state rather than a pending choice, point at `report`.
