Close out the current branch by updating the spec and creating the final commit.

> **Abandoned instead of shipped?** If the branch's work is being dropped because the spec was
> **superseded** (a different approach won), don't invent a status. Close the spec as **`done`**
> with a **SUPERSEDED header** (per the `spec-conventions` skill) *instead of* `## Implementation
> Notes`, note it on the PR, and skip the change-division report. The rest of this command covers
> the normal shipped case.

1. Identify which spec corresponds to the current branch (check for a `doing` spec whose branch reference matches the current branch, or ask the user).
2. Read the spec and compare it against what was actually implemented (read relevant source files, git log).
3. Update the spec:
   - Rename using `git mv` so the rename is tracked:
     ```
     git mv specs/NNN-doing-slug.md specs/NNN-done-slug.md
     ```
   - Add the `## Implementation Notes` section (defined in the `spec-conventions` skill) describing how the implementation differed from the spec — decisions made during coding, things deferred, things that changed
   - **Change-division note (inside `## Implementation Notes`):** assess how this branch divided the work into commits and PRs against the project's `CONTRIBUTING.md` (the authority — don't impose your own rules), and record it. This is a **report, not a fix** — do **not** restructure commits or rewrite history. State briefly how the split was done (commits / pre-PR / stack) and any divergence from `CONTRIBUTING.md`; if it conforms, one line saying so is enough. No `CONTRIBUTING.md` → skip.
   - Confirm the branch and Linear ticket are referenced at the top
   - **API Diff subsection:** check the project's `CLAUDE.md` for an `## API Modules` section:
     - **Section absent:** ask the user "Does this project have any API modules — modules whose compiled artifact is consumed as a library by another service? If yes, list them. If no, just say so." Then update `CLAUDE.md` with either the listed paths or an explicit "none" before proceeding.
     - **Section present, says "none":** skip the subsection.
     - **Section present, lists paths:** check whether the branch diff touches any file under those paths. If none were touched, skip the subsection. If any were touched, add a `### API Diff` subsection inside `## Implementation Notes`. Write it final-state only — no history, no "was renamed from". Document:
       - New or changed endpoints: HTTP method, path, return type (one line each)
       - New or changed DTOs: record/class signature with field names, types, and inline comment for nullable fields
       - New enums or new values added to existing enums
       - Removed or deprecated surface (one line: what it is, what replaces it if anything)
       - Omit internal types not exported by the API module
4. **Apply every content edit to the spec first** — the `Status:`/`Branch:` header and the full
   `## Implementation Notes` section — **then** stage and commit as the last thing you do. Stage
   **only the `-done-` path**: the `git mv` already staged the old path's deletion, so naming the
   now-nonexistent `-doing-` path makes `git add` abort with `fatal: pathspec … did not match any
   files` — which stages **nothing** (git's add is all-or-nothing on a bad pathspec) and strands
   your header + Implementation Notes edits unstaged, leaving a rename-only finish commit. Do **not**
   list the old path:
   ```
   git add specs/NNN-done-slug.md          # ONLY the -done- path; the -doing- path no longer exists
   git commit -m "<issue-id> Spec NNN: doing → done

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
   ```
5. **Verify the finish is complete before pushing** (this is the failure mode to guard against —
   a commit that captured only the rename): `git status --porcelain specs/` must be empty, and
   `git show --stat HEAD` must show insertions on `specs/NNN-done-slug.md`. A stat line reading
   `{doing => done} | 0` with `0 insertions` means the header + Implementation Notes were stranded
   — re-run `git add specs/NNN-done-slug.md` and `git commit --amend --no-edit` (or, if already
   pushed, add a follow-up commit; never force-push a shared branch).
6. Remind the user to push and open a PR.
