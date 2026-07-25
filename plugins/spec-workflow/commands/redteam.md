Adversarially stress-test a spec (or a set) for **gaps and inconsistencies in the document
itself** — the things that would stall or mislead an implementer. Report-only: it never edits,
never changes status, never opens anything. Usage: `/spec-workflow:redteam <NNN|slug> [more…]`, or
no argument to red-team every `doing` spec.

> **Hard rules.**
> - **Specs only, `todo` or `doing` only.** Skip concerns (exploratory by design) and `done` specs
>   (already built) with a one-line reason. If nothing valid remains, say so and stop.
> - **Document analysis only.** Read the target spec(s) and any spec they *directly reference*
>   (blocked-by / depends-on / supersedes) to check cross-consistency. **Never read the codebase** —
>   this judges the spec as written, not the implementation.
> - **Never fabricate.** Every finding must cite specific spec text (or a specific, named omission).
>   If you can't point at it, don't report it. Padding a report with invented concerns is a failure.
> - **A clean spec is a valid result.** If it holds up, say "no material gaps found" and stop.
> - **Surface, don't solve.** Name the gap and the question the author must answer — do **not**
>   prescribe the fix or act on it.

The spec structure and section contract come from the **`spec-conventions`** skill
(`${CLAUDE_PLUGIN_ROOT}/skills/spec-conventions/SKILL.md`) — read it so you know what a complete
spec is supposed to contain.

## 1. Resolve and gate the targets

Identify the spec file(s) from the argument (or all `doing` specs if none given). For each, confirm
it is a **spec** with status **`todo`** or **`doing`**. Drop anything else, stating why.

## 2. Read

Read each surviving target **in full**, plus every spec it directly references (for cross-checking
only). Do not open source files, tests, or configs.

## 3. Attack it — through these lenses

Look for real problems, each of which must be evidenced by the text:
- **Buildability gap** — the `Decision`/`Implementation` omits something an implementer must know:
  undefined data shapes, unspecified behavior on an obvious path, an edge case the Decision implies
  but never addresses.
- **Internal contradiction** — `Decision` vs `Implementation` vs `Known Gaps` disagree.
- **Undecided-as-decided** — an open choice written as settled; hand-waved "somehow" / "TBD" sitting
  inside what claims to be a decision.
- **Unstated assumption / dependency** — relies on a system, precondition, or another spec that is
  never named (or is named but contradicts this spec).
- **Untestable acceptance** — no clear, checkable "done" condition; success can't be verified.
- **Scope inconsistency** — `Known Gaps` contradicts the `Decision`, or the spec quietly grows
  beyond its stated goal.
- **Cross-spec conflict** — contradicts a spec it depends on or supersedes.

## 4. Report

Rank findings by severity:
- **Blocker** — would stall the build or send it the wrong way.
- **Risk** — likely rework or a wrong assumption, but buildable.
- **Minor** — small ambiguity worth a sentence.

For each finding give: the **lens**, the **citation** (quote the spec line or name the missing
section), **why it's a gap**, and the **question the author should answer**. No solutions, no edits.

Close with a one-line verdict per spec — e.g. *"002: 1 blocker, 2 risks — not ready to build"* or
*"005: no material gaps found."* Nothing is written or changed.
