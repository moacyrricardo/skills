Audit the specs catalog for **structural integrity** — not content quality. Cross-check
`CATALOG.md` against the spec files, catch number collisions, and validate the cross-reference
graph between specs. Produces a report + a concrete fix proposal, then applies the mechanical
fixes on your confirmation.

> This is a **knowledge-graph** check: is the catalog internally consistent? It never evaluates
> whether a spec is well-written — that's `redteam`'s job. The naming, statuses, `CATALOG.md`
> format, and cross-reference conventions all come from the **`spec-conventions`** skill
> (`${CLAUDE_PLUGIN_ROOT}/skills/spec-conventions/SKILL.md`) — read it first; it defines what
> "consistent" means here.

## 1. Gather ground truth

- List `specs/` and parse every `NNN-status-slug.md` into `(NNN, status, slug)`. **The filename is
  authoritative** for a document's status.
- Read `specs/CATALOG.md` if it exists. If it doesn't, note that and run only the file-level checks
  (§2–§3, §5); skip the CATALOG cross-check (§4).

## 2. Filename well-formedness

For each file under `specs/`:
- Matches `NNN-status-slug.md` with a zero-padded `NNN`.
- `status` is a valid lifecycle state (`todo` / `doing` / `done`). Concerns follow the same naming.
- Flag anything malformed (bad number padding, unknown status token, stray files).

## 3. Number collisions

Group files by `NNN`. Any number owned by **more than one** file is a collision — report each
colliding set. (Collisions usually need a human to decide which keeps the number, so these go to
the proposal as *needs-decision*, not auto-fix.)

## 4. CATALOG.md ↔ files

Only if `CATALOG.md` exists:
- **Stale rows** — a row whose `NNN`/filename doesn't resolve to a real file (renamed or removed).
- **Orphan files** — a spec file with **no** row in the catalog.
- **Status/slug drift** — a row whose status or slug disagrees with the filename. The filename
  wins; the row is what's wrong.

## 5. Cross-reference graph

Scan every spec/concern body for references to other documents and validate each edge:
- Patterns: `blocked by NNN` / `blocked-by: NNN` / `depends on NNN`, `resolved by (spec) NNN`,
  `supersedes NNN`, and the concern **WARNING** block's `resolved by spec NNN` pointer.
- For each edge check: **target `NNN` exists**, and its **status is sane** for the relation, e.g.:
  - a concern marked "resolved by NNN" whose target is missing, or is itself a concern → broken.
  - `blocked by NNN` where `NNN` is already `done` → stale block (note; the blocker cleared).
  - `supersedes NNN` where `NNN` lacks the reciprocal WARNING/`done` state → asymmetric edge.
- Report each broken or stale edge with the **source file**, the **quoted reference**, and why.

## 6. Report

Group findings by severity:
- **Error** — the graph is broken (dangling reference, collision, stale/missing CATALOG row).
- **Warning** — inconsistent but not broken (status drift, stale block, asymmetric edge).
- **Info** — cosmetic (padding, ordering).

Each finding: `specs/<file>` + the exact offending text + one line on what's wrong. Be terse.

## 7. Fix proposal + apply

Turn the findings into a concrete plan, marking each item:
- **auto** — mechanical and unambiguous: rewrite a CATALOG row to match its filename, add a missing
  row, drop a stale row, correct an obvious cross-ref typo (`003`→`030` when only `030` exists).
- **needs-decision** — requires human judgement: NNN collisions (which file renumbers?), a dangling
  reference with no clear target, an asymmetric supersede.

Show the plan, then ask: **"Apply the `auto` fixes? (the `needs-decision` items stay for you.)"**
On confirmation, apply **only** the `auto` items — edit `CATALOG.md` and correct cross-references.
**Never** touch a spec's semantic content (Context/Decision/Implementation/…); this command only
repairs the graph, not the prose. Report exactly what was changed.
