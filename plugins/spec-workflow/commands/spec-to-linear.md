Set up a spec for implementation: find or create a Linear issue, mark the spec as doing on main, then checkout the feature branch.

The todo → doing rename is committed to `main` first so the catalog always reflects in-progress work accurately. The feature branch is created from the updated main.

---

## 1. Identify the spec(s)

- If the user referenced a spec by number or slug, read `specs/NNN-*-slug.md`.
- If no spec is referenced but the user described a task, confirm with them before proceeding — ask which spec applies or whether they want to proceed without one.
- If the spec is already `doing` or `done`, warn the user and ask whether to continue.

## 2. Ensure we are on main

```
git branch --show-current
```

If not on `main`, switch:
```
git checkout main && git pull
```

If the working tree is dirty, stash first and tell the user:
```
git stash push -m "auto-stash before spec-to-linear"
```

## 3. Resolve Linear team

Look in the project's `CLAUDE.md` for a line like `Linear team: <key>`.

If missing:
- Ask the user for the team key or name.
- After they confirm, append `Linear team: <key>` to `CLAUDE.md`.

## 4. Find or create the Linear issue

**First**, check whether there is already a known Linear issue ID for this spec. Look at the spec file and any context the user provided.

If a branch name with an issue ID exists (format `[A-Z]{3}-[0-9]+`), look it up directly:
```
linearis issues get <ID>
```
If the issue exists, confirm with the user that it is the right one.

Otherwise, fall back to a title search:
```
linearis issues search "<spec title or task summary>" --team <team>
```

If a match is found, confirm with the user before using it.

If no match, create a new issue:

1. Get the active cycle:
   ```
   linearis cycles list --team <team> --active
   ```
2. Get available labels:
   ```
   linearis labels list --team <team>
   ```
3. Infer labels from spec or task content. Only apply labels that clearly fit — if unsure, ask.
4. Create the issue:
   ```
   linearis issues create "<title>" \
     --team <team> \
     --cycle <active-cycle-id> \
     --labels <comma-separated> \
     --description "<one-paragraph summary from spec or task>"
   ```

Note the returned issue ID (e.g. `ABC-123`).

Use the branch name exactly as returned by Linear in the `branchName` field of the issue.
Do not shorten, clean up, or derive your own slug — always use Linear's branch name verbatim.

## 5. Rename the spec and commit to main

For each spec file involved:
- Rename using `git mv` so the rename is tracked:
  ```
  git mv specs/NNN-todo-slug.md specs/NNN-doing-slug.md
  ```
- Add at the very top of the file (before the `# heading`):
  ```
  Branch: <linear-branchName>   # Linear's verbatim branchName, e.g. alex/abc-123-short-slug
  Linear: <issue-id>            # e.g. ABC-123
  ```
- Commit and push to main:
  ```
  git add specs/NNN-todo-slug.md specs/NNN-doing-slug.md
  git commit -m "<issue-id> Spec NNN: todo → doing"
  git push origin main
  ```

This commit on `main` makes the in-progress status visible in the catalog immediately.

## 6. Checkout the feature branch

Create the feature branch from the now-updated main:
```
git checkout -b <linear-branchName>   # the verbatim branchName from step 4
```

If a stash was made in step 2, restore it now:
```
git stash pop
```
Tell the user their uncommitted changes have been restored.
