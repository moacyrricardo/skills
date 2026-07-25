Squash-merge one PR or a chain of stacked PRs into main, handling the rebase bookkeeping between each merge.

**Input:** Space-separated PR numbers in merge order, e.g. `3 5 6 7` or a single number `3`.

---

## 1. Gather PR info

For each PR number provided, fetch:

```
gh pr view <N> --json number,title,headRefName,baseRefName,headRefOid,state
```

Record for each PR:
- `state` — must be `OPEN`; stop and report any that are not
- `headRefName` — the branch being merged (e.g. `alex/abc-806-create-integration-flow`)
- `baseRefName` — what it currently targets (its parent in the stack or `main`)
- `headRefOid` — the HEAD SHA **before** merge; this is the `--onto` cut point for rebasing the next branch
- `title` — for display and squash commit subject

Derive the squash commit subject from the branch name: extract the issue id from `headRefName` (the segment matching `[A-Z][A-Z0-9]+-\d+`, e.g. `ABC-806`) and combine with the PR title, stripped of any leading `TAG: ` prefix your tracker/convention adds (e.g. `LIN: `). If the branch has no issue id, just use the cleaned PR title.

Example: branch `alex/abc-806-create-integration-flow`, title `LIN: Create integration flow` → subject `ABC-806 Create integration flow`.

---

## 2. Build the execution plan

Work through the PR list in order. For each PR at position i:

**A. Squash-merge**
```
gh pr merge <N> --squash --delete-branch --subject "<derived subject>"
```

**B. Sync local**
```
git fetch origin
```

**C. Rebase the next PR's branch onto main** (only if there is a next PR AND its `baseRefName` matches the current PR's `headRefName`)

```
git checkout <next-headRefName>
git rebase --onto origin/main <current-headRefOid> HEAD
git push --force-with-lease origin <next-headRefName>
gh pr edit <next-N> --base main
```

The `<current-headRefOid>` is the SHA recorded in step 1 — the tip of the current branch before it was squash-merged. This tells git exactly where to cut: "replay only the commits above this point".

If the next PR's `baseRefName` does NOT match the current PR's `headRefName` (e.g. it was already rebased or independently targets main), skip the rebase block for that transition.

---

## 3. Present the plan

Print all commands grouped by PR step, with clear labels. Example format:

```
=== Step 1 — PR #3: ABC-786 Google login flow ===
  gh pr merge 3 --squash --delete-branch --subject "ABC-786 Google login flow"
  git fetch origin

  → Rebase PR #5 (ABC-792) onto main
  git checkout alex/abc-792-integration-entity
  git rebase --onto origin/main <headRefOid of ABC-786> HEAD
  git push --force-with-lease origin alex/abc-792-integration-entity
  gh pr edit 5 --base main

=== Step 2 — PR #5: ABC-792 Integration entity ===
  gh pr merge 5 --squash --delete-branch --subject "ABC-792 Integration entity"
  git fetch origin

  → Rebase PR #7 (ABC-806) onto main
  ...

=== Final ===
  git checkout main
  git pull
```

Show the actual SHA values (not placeholders) for the `--onto` arguments.

Ask: **"Plan looks good? Confirm to execute."**

Wait for confirmation before running anything.

---

## 4. Execute

Run each command in sequence. After every rebase step, show:
```
git log --oneline origin/main..HEAD
```
so the user can verify only the right commits are being carried forward before the next merge.

If any command fails, stop immediately, show the error, and do not proceed.

After all PRs are merged, run `git checkout main && git pull` and report the list of merged PRs with their squash commit SHAs.
