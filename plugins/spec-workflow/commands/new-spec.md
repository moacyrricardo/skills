Help the user create a new spec or concern file in the project catalog.

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
     git stash push -m "auto-stash before new-spec"
     ```
     Tell the user: "Stashed your uncommitted changes — will restore them after creating the spec."
  3. Switch to main and pull:
     ```
     git checkout main && git pull
     ```
  4. Remember the original branch name so you can switch back at the end.

## 1. Read the `specs/` directory to determine the next available number (NNN).

## 2. Ask the user: is this a **concern** (exploratory, options open) or a **spec** (decision made, ready to build)?

## 3. Ask for a short slug (2-4 words, hyphenated).

## 4. Create the file as `specs/NNN-todo-slug.md` with this structure:

For a **concern**:
```
# NNN — Title

## Problem
[What is the issue or tension being explored?]

## Hypotheses / Options
[List options with brief pros/cons]

## Open Questions
[What needs to be decided before this can become a spec?]
```

For a **spec**:
```
# NNN — Title

## Context
[Why this work is being done]

## Decision
[What was decided]

## Implementation
[Technical detail: models, methods, migrations, APIs]

## Known Gaps
[What is deliberately out of scope or deferred]
```

## 5. Commit and push to main

```
git add specs/NNN-todo-slug.md
git commit -m "Spec NNN: add <slug>"
git push origin main
```

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

Tell the user the spec was created on `main` as `specs/NNN-todo-slug.md` and is ready for `/spec-workflow:spec-to-linear` when they're ready to start building.
