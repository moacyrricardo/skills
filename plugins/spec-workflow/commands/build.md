Implement a spec end-to-end: code, tests, and hand-off to /spec-workflow:finish. Calls /spec-workflow:start first if the branch and Linear issue are not already set up.

---

## 1. Check prerequisites

Determine whether `/spec-workflow:start` has already been run by checking:
- Is there a `doing` spec in `specs/` whose branch matches the current git branch?
- Is the current branch a feature branch (not `main`/`master`/`develop`)?

If both are true, skip to step 2.

If not, and the context is clear enough (spec identified, team known), run `/spec-workflow:start` now.

If context is ambiguous — no spec referenced, team unknown, or branch state is unclear — ask the user before proceeding.

## 2. Read the spec

Read the `doing` spec thoroughly. Its structure and section names follow the `spec-conventions`
skill (`${CLAUDE_PLUGIN_ROOT}/skills/spec-conventions/SKILL.md`). Understand:
- What is being built and why (Context + Decision sections)
- The technical detail (Implementation section)
- What is explicitly out of scope (Known Gaps)

If the user described a task without a spec, use that description as the implementation guide.

## 3. Implement

Divide the work into commits and PRs as the project's `CONTRIBUTING.md` prescribes — it is the source of truth for how a change is split (what belongs in a commit, what is an enabling refactor that lands first, and what goes in the PR vs a pre-PR vs a deferred follow-up). Plan that division **before** writing code and reflect it in the spec's `## Implementation`. If the project has no `CONTRIBUTING.md`, fall back to the commit conventions in its `CLAUDE.md`.

When a decision point arises that is not covered by the spec, make the call and note it — do not stop to ask unless the ambiguity is blocking.

## 4. Run tests

Infer the test runner from the project — check in order:
- `package.json` `scripts.test` field
- `Makefile` targets (`test`, `check`)
- `pytest.ini` / `pyproject.toml` (Python)
- `go test ./...` (Go)
- `CLAUDE.md` if it documents the test command

Run the full test suite. Fix all failures before proceeding.

## 5. Add regression tests

Write tests that cover the new behavior introduced in this implementation. These should fail if the new behavior is accidentally removed later. Follow the existing test conventions in the project.

## 6. Confirm and hand off

Present a summary:
- What was implemented (brief, by logical area)
- List of commits made
- Tests added or modified
- Any decisions made that deviated from or were not covered by the spec

Ask: **"Ready to call /spec-workflow:finish. Confirm to proceed?"**

Wait for confirmation. On confirmation, invoke `/spec-workflow:finish`.
