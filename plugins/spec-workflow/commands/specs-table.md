Produce a planning table from the specs catalog.

## Default scope

Unless the user explicitly asks for something different, only include **actionable work**:
- `todo` specs (decided, not started)
- `doing` specs (in progress)
- `concern` specs that are **not** marked as resolved (no `✅` in the catalog row and no WARNING block at the top of the file pointing to a resolving spec)

Skip `done` specs and resolved concerns unless the user asks for them (e.g. "all", "include done", "full history").

## Steps

1. Read `specs/catalog.md` to get the full list and quickly identify the subset to process.

2. For each spec in scope, read the file and extract:

   - **#** — the NNN number
   - **Summary** — one tight sentence: what is being built / what problem is being solved
   - **Open questions / blockers** — things that are still undecided or that block implementation:
     - For `concern` files: items listed under `## Open Questions` or `## Open Gaps` that have no resolution note
     - For `todo`/`doing` specs: items in `## Known Gaps`, open `#comment:` lines, or explicit "deferred" notes that affect the current scope
     - If nothing is open, write "—"
   - **Ease** — implementation size estimate. Use one of: `XS · trivial`, `S · small`, `M · medium`, `L · large`, `XL · complex`. Base this on:
     - Number of layers touched (entity, use case, API, migration, tests)
     - Whether the spec is fully detailed or leaves design work to the implementer
     - Dependencies on other unfinished specs (add "⚠ blocked by NNN" if hard-blocked)

3. Render as a markdown table:

```
| # | Summary | Open questions / blockers | Ease |
|---|---------|--------------------------|------|
| 039 | … | … | M · medium |
```

4. After the table, add a brief **Notes** section (2–5 bullets) calling out:
   - Any hard dependencies between the specs in the table (e.g. "043 should go before 039 because…")
   - Specs that are disproportionately high-leverage (unblocks several others)
   - Concerns that are close to resolution and could quickly become a spec

## Output tone

Be concise. The table is a planning aid — the reader knows the project. No preamble paragraphs.
