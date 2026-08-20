---
description: Synthesize a readable, self-contained interactive HTML report from heterogeneous sources — docs, issue trackers, PRs, git history, code, logs. Asks what angle and audience you need, gathers and cross-checks the sources, then emits one theme-aware file with the conclusion up top and the evidence one expand away. Read-only — it never changes the things it reports on.
argument-hint: <what to report on> [angle/audience notes]
---

Build a rich HTML **report** about **$ARGUMENTS**.

A report turns scattered, heterogeneous inputs into **one readable document**. It is a document
*about* things — it reads and synthesizes, it **never mutates** the sources (no commits, no issue
edits, no pushes). The output is a single self-contained interactive HTML file.

**Format contract:** the *how* of the file — self-containment / Artifact CSP, theme-awareness, and
the progressive-disclosure (expandable-context) pattern — comes from the **`html-doc`** skill. Read
`${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first and conform to it. This command owns only the
*what*: framing, gathering, and structuring the content.

## 1. Frame it — angle and audience first (don't skip)

A report with no chosen angle becomes an undifferentiated data dump. Before gathering, settle:

- **Audience** — who reads this (you, a teammate, a stakeholder), which sets depth and vocabulary.
- **Angle / question** — the one thing the report is *for*: "why did this regress?", "what changed
  this week?", "is this spec done?", "what's the state of X?". Everything included must serve it.
- **Sources in scope** — which of the available inputs are relevant (see §2).

If `$ARGUMENTS` already makes these clear, state your reading and proceed. If a load-bearing choice
is genuinely ambiguous, ask **one** concise question — then proceed.

## 2. Gather — pull from whatever sources fit, and cross-check

Draw only from sources actually present in this environment; skip the rest without ceremony:

- **Repo & git** — code, `git log`/`git diff`, blame, file structure.
- **PRs / GitHub** — `gh pr view`, review threads, checks, changed files.
- **Issue tracker** — the project's tracker CLI if its `CLAUDE.md` documents one (e.g. `linearis`),
  for issue state, threads, and cross-references.
- **Docs & specs** — `README`, `ARCH.md`, `CONTRIBUTING.md`, a `specs/` catalog if present.
- **Logs / observability** — error trackers or log excerpts when the angle is a failure.

**Cross-check across sources rather than trusting one.** When two disagree (a PR says merged, the
tracker says open), that discrepancy is itself a finding — surface it, don't silently pick one.
Read enough to be right; a report that misreads its sources is worse than none.

## 3. Structure — conclusion first, evidence one expand away

Lead with the answer to the framing question; make the reader *choose* to go deeper:

- **Headline / summary** at the very top — the conclusion, the state, the bottom line in a few
  sentences. A reader who stops here should still get the point.
- **Sections** (`<h2>`/`<section>`) for the substantive parts, ordered by what the audience needs
  first, not by which source you happened to read first.
- **Evidence lives in `<details>`** — each claim that rests on a source carries its proof one
  expand away (the diff hunk, the issue excerpt, the log line, the changed-files list), collapsed
  by default per the `html-doc` progressive-disclosure rule. Escape interpolated source text.
- **Attribute claims** to their source (link the PR/issue, name the file:line) so the report is
  checkable, not just assertable.
- Back a claim with the real artifact, not just a description of it — a one-line claim *plus* its
  supporting form from `html-doc`'s **Visual evidence** section: a fenced diff or table, a chart for
  a trend (load `dataviz`), an inline-SVG diagram for a flow, an existing screenshot for a UI state.
  Charts are the form a report reaches for most. The prose still carries the point; the artifact
  substantiates it.

## 4. Deliver

Write the `.html` to a sensible path (the user's scratchpad unless they name a location) and hand
it back with the file tools; offer to publish it as an Artifact if they want a shareable link.
Give a one-or-two-line summary of what the report concludes — don't restate the whole thing in
chat; the file is the deliverable.

## Notes

- **Read-only** is a hard guarantee: this command inspects sources and writes exactly one output
  file. It never commits, edits an issue, pushes, or otherwise changes what it reports on.
- Project-agnostic: every source-specific detail (which tracker, which build) comes from the target
  project's `CLAUDE.md`, never hardcoded here.
- If you find a *decision that needs making* rather than a fact to report, that's the `decide`
  command's job, not this one — note it and point there.
