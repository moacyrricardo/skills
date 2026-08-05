Help the user create a new spec or concern file in the project catalog.

**Model & templates:** the definitions of *concern* vs *spec*, the `NNN-status-slug` naming, and
the per-status file templates all come from the **`spec-conventions`** skill — read
`${CLAUDE_PLUGIN_ROOT}/skills/spec-conventions/SKILL.md` first and create files that conform to it.
This command is only the *create-on-`main`* procedure.

Specs are always created on `main` so the catalog is a faithful view of status.

## 0. Ensure we are on main

Check the current branch:
```
git branch --show-current
```

- **If already on `main`**: proceed.
- **If on another branch**:
  1. Check for uncommitted changes: `git status --porcelain`
  2. If the working tree is dirty, stash first:
     ```
     git stash push -m "auto-stash before new"
     ```
     Tell the user: "Stashed your uncommitted changes — will restore them after creating the spec."
  3. Switch to main and pull:
     ```
     git checkout main && git pull
     ```
  4. Remember the original branch name so you can switch back at the end.

## 1. Read the `specs/` directory to determine the next available number (NNN).

## 2. Ask the user: **concern** or **spec**? (The distinction is defined in the `spec-conventions` skill.)

## 3. Ask for a short slug (2-4 words, hyphenated).

## 4. Create the file with the status token that matches the kind

The **status is part of the filename** and must match what the user picked in step 2 (the closed
status set lives in the `spec-conventions` skill):
- **concern** → `specs/NNN-concern-slug.md`
- **spec** → `specs/NNN-todo-slug.md` (a new spec is decided but not yet started)

This matters: the filename is the *authoritative* status, so a concern mistakenly named
`NNN-todo-slug.md` reads as a buildable spec — `plan` lists it as actionable and `autopilot`'s
readiness gate stops auto-skipping it. A concern is `concern`, not `todo`.

Use the matching **concern** or **spec** template from the `spec-conventions` skill (its
`## File templates` section) and fill each section for this document. The section headings there
are a contract other commands read — use them verbatim; don't rename or invent your own.

## 5. Commit and push to main

```
git add specs/NNN-<status>-slug.md
git commit -m "Spec NNN: add <slug>"
git push origin main
```
(`<status>` is `concern` or `todo`, matching the filename from step 4.)

## 6. Return to the original branch (if we switched)

If we stashed and switched:
```
git checkout <original-branch>
git stash pop
```
Tell the user: "Back on <original-branch> with your uncommitted changes restored."

If no stash was needed but we switched:
```
git checkout <original-branch>
```

## 7. Confirm

Tell the user what was created on `main`:
- a **spec** (`specs/NNN-todo-slug.md`) is ready for `/spec-workflow:start` when they want to begin building it.
- a **concern** (`specs/NNN-concern-slug.md`) stays exploratory — it is later *resolved by* a spec (which adds the WARNING block to it), not built directly.
