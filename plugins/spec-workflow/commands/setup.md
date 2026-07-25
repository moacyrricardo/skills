Preemptively configure the spec workflow for **this** project. Read what the spec commands and
agents actually need, inspect the codebase and existing `CLAUDE.md`, **infer** what can be
determined safely, **ask** for the rest, and write the result into `CLAUDE.md` so later commands
never have to stop and ask. Idempotent — re-run any time to reconcile.

> Nothing here is project-specific in the plugin; the commands read every project value from the
> target repo's `CLAUDE.md`. This command's job is to populate that file correctly, once.

## 1. Derive the config set (what the commands need)

Read this plugin's own commands and agents to enumerate every value they read from a project's
`CLAUDE.md` — so the set stays in sync with the commands, not a hardcoded guess:

```
ls ${CLAUDE_PLUGIN_ROOT}/commands ${CLAUDE_PLUGIN_ROOT}/agents
grep -rniE 'CLAUDE\.md|Linear team|Dev server|Build & test|branch|API Modules|dev-login' ${CLAUDE_PLUGIN_ROOT}
```

The baseline set (confirm against what you find):
- **Tracker + team key** — e.g. `Linear team: <key>` (used by `start`, `ship`). Optional if no tracker.
- **Dev server** — start command, port, readiness signal (used by `run-local-dev`, `test-live`).
- **Build & test command** — how to build and run tests for one module (used by `build`, `autopilot`).
- **Branch pattern** — how feature branches are named (tracker branchName, or a fallback pattern).
- **Specs dir** — where the catalog lives (default `specs/`).
- **CONTRIBUTING.md** — present? It's authoritative for commit/PR division if so.
- **API modules** — paths whose build artifact another service consumes (used by `finish`).
- **Dev-login recipe** — how to authenticate the running app for evidence capture, if any.

## 2. Inspect the project — infer what you can

Determine each value from evidence, and record the evidence + a confidence:
- **Repo / PR flow** — `git remote -v`, `gh repo view` → GitHub remote present?
- **Build & test / dev server** — detect from the ecosystem: `package.json` scripts, `Makefile`,
  `pyproject.toml` / `tox.ini`, `pom.xml`, `Cargo.toml`, `go.mod`, etc. Read the actual scripts;
  don't assume. A dev-server port often lives in the framework config — propose it, flag it as a guess.
- **Specs dir / CONTRIBUTING** — does `specs/` exist? `CONTRIBUTING.md`?
- **Existing `CLAUDE.md`** — read it. Any value already set is kept; you're filling gaps and
  reconciling, never clobbering unrelated content.

## 3. Classify each value

Split the config set into:
- **Inferred** — determined with evidence (state the value + where it came from).
- **Needs-ask** — can't be determined from the repo: the tracker + team key, a dev-server port you
  couldn't find, an ambiguous build command, the dev-login recipe.

## 4. Confirm and ask

Show the **inferred** table for a quick confirm ("looks right? correct anything"), then ask **only**
the needs-ask items — batched into one round, not one at a time. If the project has no issue tracker,
let the user say so and drop the tracker fields entirely (the workflow is tracker-agnostic).

## 5. Write to `CLAUDE.md`

Create or update the workflow-config sections — only those sections, leaving everything else intact:

```markdown
## Spec workflow
Tracker:       linear | none
Linear team:   <key>          # omit if no tracker
Branch prefix: <pattern>
Specs dir:     specs/

## Dev server
Start: <command>
Port:  <port>
Ready: <readiness signal>
```

(Add `Build & test`, `API Modules`, or a dev-login note if the project needs them.) Then report a
short summary: what was inferred, what you asked, and exactly what was written to `CLAUDE.md`.
