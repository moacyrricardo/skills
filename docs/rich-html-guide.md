# rich-html — turn your work into interactive HTML deliverables

A short guide to the `rich-html` plugin and its two commands, `report` and `decide`. Both take the
scattered inputs of your work and turn them into a single **self-contained, interactive HTML file** —
one to *read*, one to *act on*.

> 📖 **Prefer the visual version?** Open [`rich-html-guide.html`](./rich-html-guide.html) in a
> browser for the same guide with the two-track diagram and nicer formatting. This Markdown page is
> the quick, GitHub-readable companion.

```
sources    → report → a readable document
decisions  → decide → an expandable surface + one prompt
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

Once installed the commands are namespaced: `/rich-html:report`, `/rich-html:decide`.

## Two tools, one system

Both commands produce a **document about your work** — not the product itself (that's what the
`prototype` plugin's `mockup` is for). They share a format contract, the `html-doc` skill, so the
two outputs feel like one system.

| | `report` | `decide` |
| :-- | :------- | :------- |
| **Turns** | heterogeneous sources | pending decisions |
| **Into** | a readable document | an interactive decision surface |
| **You** | read it | choose per decision, then run the emitted prompt |
| **For** | facts *about* things | the forks *between* things |

## The shared format — the `html-doc` skill

`report` and `decide` both build on `html-doc`, which encodes the three non-obvious rules that make
one HTML file work everywhere and stay readable:

- **Self-contained** — inline CSS/JS, no network, assets as `data:` URIs. Renders identically as a
  local file, an email attachment, or a Claude Artifact (which enforces a strict CSP).
- **Theme-aware** — respects the reader's light/dark preference with a manual toggle. (A *document*
  adapts to the reader — the opposite of a `mockup`, which matches the product's real theme.)
- **Progressive disclosure** — the conclusion is visible; the evidence is one expand away, in place,
  via native `<details>`. Dense, but never a wall.

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

## When to use which

- Reaching for the **state of something**, a synthesis, a briefing → **`report`**.
- Facing a pile of **unmade choices** and want to clear them in one pass → **`decide`**.

If `report` runs into a decision that needs making, it points at `decide`; if `decide` finds
reportable state rather than a fork, it points back at `report`.

---

Part of [moacyrricardo/skills](https://github.com/moacyrricardo/skills) · see the
[README](../README.md) for the full plugin reference.
