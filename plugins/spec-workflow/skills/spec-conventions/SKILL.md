---
name: spec-conventions
description: The shared model for spec-driven development — what a spec vs a concern is, the NNN-status-slug naming and todo→doing→done lifecycle, the section templates for each, and the specs/ catalog with its CATALOG.md index. Load this when creating, reading, evaluating, renaming, resolving, or cataloging specs and concerns, or when maintaining CATALOG.md. This is the single source of truth the other spec-workflow commands and the autopilot agent build on — prefer it over any inline restatement.
---

# Specs, concerns, and the catalog

This is the vocabulary and file contract the whole `spec-workflow` plugin is built on. Every
command and the `autopilot` agent operate on the model defined here; when they mention a spec,
a concern, a status, or a section name, this is what they mean.

## Concern vs spec

The `specs/` directory is the project's **architectural decision catalog** — a numbered, durable
record of two kinds of document:

- **Concern** — an *exploratory* document. It frames a problem, enumerates hypotheses and
  trade-offs, and keeps options open. **It is not a decision.**
- **Spec** — a *decision made*. Technically detailed and prescriptive, ready to hand to whoever
  will build it.

A concern that gets resolved is superseded by a spec. When that happens, add a **WARNING** block
at the top of the concern pointing to the resolving spec (by number) and noting what is now
invalid — never delete the concern; its history is part of the record.

## Naming

Every file is `NNN-status-slug.md`:

- **`NNN`** — a permanent, zero-padded number (`001`, `042`). Assigned once from the next free
  number in `specs/` and **never reused or renumbered** — it is the stable cross-reference id.
- **`status`** — the lifecycle state (below). Status changes are made by **renaming the file**
  (`git mv`), so the filename is the authoritative status of a document.
- **`slug`** — 2–4 hyphenated words, human-readable.

The file's first heading mirrors it: `# NNN — Title`.

## Lifecycle

**Specs** move through three states, each an isolated rename committed as its own step:

```
todo  ──▶  doing  ──▶  done
(decided,  (being      (shipped; final
 not       built on    commit updates the
 started)  a branch)   spec)
```

- `todo → doing` is committed to `main` first, so the catalog always reflects in-progress work.
- A `doing`/`done` spec carries a small metadata header above the `#` heading:
  ```
  Status: doing
  Branch: <the feature branch>
  Issue:  <issue-id>        # your tracker's id, if you use one; omit otherwise
  ```
- `doing → done` is the final commit on the branch: `git mv` the file, flip `Status:`, keep the
  `Branch:` ref, and add the `## Implementation Notes` section (see below).

**Concerns** don't run `doing`/`done`. They live as `todo` while open and are retired by adding
the WARNING block when a spec resolves them.

**The status set is closed.** The only statuses are `todo`, `doing`, `done` (specs) and `concern`.
**Never invent a new token** (`closed`, `wontfix`, `blocked`, `archived`, …). Any other state a
document is in belongs in its *body*, not its filename.

**A spec that will not be built** — superseded by a different approach, or abandoned — is still
closed out as **`done`** (never a new status), with a **SUPERSEDED header** at the very top,
mirroring how a concern is retired:

```markdown
> **SUPERSEDED — not implemented.** Replaced by spec NNN. <one line: why this approach was dropped.>
```

It stays `done` so `plan` drops it from actionable work; the header — plus a `superseded → NNN`
note in its `CATALOG.md` row — records that it shipped nothing.

## File templates

These section names are a **contract** — other commands read and write them by exact name, so use
them verbatim. Add sections beyond these freely; don't rename these.

### Concern

```markdown
# NNN — Title

## Problem
What issue or tension is being explored?

## Hypotheses / Options
Each option with brief pros/cons.

## Open Questions
What must be decided before this can become a spec?
```

Resolved concern — prepend:

```markdown
> **WARNING — resolved by spec NNN.** <what this concern concluded / what is now invalid.>
```

### Spec

```markdown
# NNN — Title

## Context
Why this work is being done.

## Decision
What was decided.

## Implementation
Technical detail: models, methods, migrations, APIs.

## Known Gaps
What is deliberately out of scope or deferred.
```

At `done`, the finishing commit appends one more section:

```markdown
## Implementation Notes
How the build differed from the spec — decisions made while coding, things deferred, things that
changed. (Sub-sections like `### API Diff` may live here.)
```

## The catalog and CATALOG.md

The `specs/` directory **is** the catalog; each filename encodes its document's authoritative
status. `specs/CATALOG.md` is a maintained **index** over it — the at-a-glance overview and the
go-to source of truth for status across the set (and the natural aggregation point when a catalog
spans repositories).

Recommended shape:

```markdown
# Catalog

| #   | Status | Title / slug        | Branch                       | Issue    | Notes |
|-----|--------|---------------------|------------------------------|----------|-------|
| 001 | done   | auth-google-login   | you/abc-786-auth-google      | ABC-786  |       |
| 002 | doing  | integration-entity  | you/abc-792-integration      | ABC-792  |       |
| 003 | todo   | search-index        | —                            | —        |       |
| 004 | concern| caching-strategy    | —                            | —        | → resolved by 006 |
```

**Keep it synced.** The spec filenames are ground truth; update `CATALOG.md` whenever a spec is
added, renamed (status change), or a concern is resolved. If the two ever disagree, the filenames
win — reconcile `CATALOG.md` to match what's on disk.

## Assets: `NNN-assets/`

When a spec or concern needs supporting artifacts — interactive mocks, screenshots, analysis
reports, sample data — they live in a sibling folder `specs/NNN-assets/`, keyed by the document's
**permanent number**. Keying on the number (not the status) is deliberate: the folder never moves
as the spec renames `todo → doing → done`, so links to it never break.

- **Reference by relative link**, usually a header callout — e.g.
  `> **Design reference:** [`142-assets/mock.html`](142-assets/mock.html)` — and mark the
  authoritative artifact as the *source of truth* when there is one.
- **Check the assets in** with the spec. Group freely inside (`144-assets/mock-landing-assets/`).
- **Only when needed** — most specs have no assets; don't create an empty folder.
- One spec may reference another's assets (note the reuse), but the folder always belongs to its
  own number.

## Inline feedback: `#comment:`

Reviewers drop `#comment: ...` lines directly inside a spec/concern file. When asked to act on
them, adapt the document accordingly and **remove the comment lines** in the same edit.

## Cross-referencing

Refer to other documents by their number ("blocked by 003", "supersedes 004"). Because `NNN` is
permanent, these references stay valid across renames and status changes.
