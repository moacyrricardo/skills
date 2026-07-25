Verify a shipped feature against the real running app and publish the proof. Orchestrates `/live-verify:run-local-dev` (ensure the server is up) + the `live-verify:test-flow-headless` agent (drive it, capture a screenshot/gif/text), then attaches that proof to the branch's GitHub PR and to the Linear ticket if there is one.

Usage: `/spec-workflow:test-live [what to verify] [url/slug/endpoint] [<issue-id>]`. Anything omitted is inferred from the `doing`/`done` spec and the PR diff; ask if genuinely ambiguous.

---

## 1. Work out the target

- **What + how to reach it**: from the args, else the current branch's `doing`/`done` spec
  (`specs/NNN-*-*.md`) and `git diff main...HEAD`. Identify the concrete URL / slug
  (`/e/{slug}`) / endpoint and any interaction (the toggle to click, the flow to walk).
- **PR**: `gh pr view --json number,url,headRefName`. No PR → tell the user; you can still
  capture + report, just nothing to comment on.
- **Linear ticket**: the arg, else an issue id in the spec header. None → skip Linear
  (many projects deliberately skip it).
- **Artifact type**: static state → screenshot; interaction/transition → gif. Auto unless
  the user said otherwise.

## 2. Ensure the server is up

Run **`/live-verify:run-local-dev`**. If it reports failure, stop and surface that — do not proceed to
capture against a dead server.

## 3. Capture (delegate to the agent)

Spawn the **`live-verify:test-flow-headless`** agent with a precise prompt: the base URL, the exact
target URL/endpoint, the interaction steps (CSS selectors in order), the desired artifact
type, and any auth note (point it at the project's CLAUDE.md dev-server section). It returns
a **verdict** + an **absolute artifact path** (or fenced request/response text for API).

Two known snags to handle, not spin on:
- **Permissions.** The agent runs in the background and can't answer permission prompts.
  If it stalls asking for Bash approval, either add its commands to the settings allow-list
  (`geckodriver`, `~/.local/share/pw-evidence/selvenv/bin/python`, `python3` + `pip`
  for the one-time venv bootstrap, `curl`, `mkdir`, `mktemp`) and re-spawn, **or
  run the capture yourself on this (main) thread** where prompts can be answered. The latter
  is the reliable default. The agent self-heals its capture venv (Step 0: a durable
  `~/.local/share/pw-evidence/selvenv` with selenium + Pillow), so no manual setup is needed.
  (The agent writes artifacts to a per-run dir under
  `$CLAUDE_JOB_DIR/tmp`, not a fixed `/tmp/verify`, so two captures can run at once without
  clobbering each other — don't pin the allow-list to a fixed path.)
- **Browser availability.** Under a sandboxed Bash tool the Firefox/geckodriver stack
  may be killed (exit 144 / SIGSTKFLT). If the browser won't run, fall back to an
  **API/curl text proof** and note that the visual capture was unavailable here.

If the verdict is **FAIL**, stop here and report the failure with its evidence — do not
publish a "verified" comment. Ask the user how to proceed.

## 4. Publish the proof (outward writes — keep them on this thread)

### 4a. Host the image on the `verification-artifacts` orphan branch
Throwaway-index plumbing so the current branch / index / working tree are untouched:
```bash
IMG=<artifact path from the agent>                  # the .png or .gif
DEST="<spec-or-feature-slug>/$(date -u +%Y%m%dT%H%M%SZ)-<name>.${IMG##*.}"
BR=verification-artifacts
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)

blob=$(git hash-object -w "$IMG")
# Push with retry: a concurrent capture may push to $BR between our read-tree and push,
# making ours a non-fast-forward. Re-fetch the new tip and rebuild the commit on it.
for attempt in 1 2 3 4 5; do
  git fetch -q origin "$BR" 2>/dev/null || true
  export GIT_INDEX_FILE=$(mktemp -u)                # a NAME, not a file — git writes a fresh index
  parent=$(git rev-parse -q --verify "refs/remotes/origin/$BR" || true)
  [ -n "$parent" ] && git read-tree "$parent"
  git update-index --add --cacheinfo "100644,$blob,$DEST"
  tree=$(git write-tree)
  commit=$([ -n "$parent" ] && git commit-tree "$tree" -p "$parent" -m "verify: $DEST" \
                            || git commit-tree "$tree" -m "verify: $DEST")
  rm -f "$GIT_INDEX_FILE"; unset GIT_INDEX_FILE
  git push -q origin "$commit:refs/heads/$BR" && break
  echo "push race on $BR (attempt $attempt) — refetching tip"; sleep 2
done

# Embed URL for the comment. Use a repo-RELATIVE blob link with ?raw=true — it renders inline
# for PRIVATE repos too. (The public raw.githubusercontent.com host 404s without a token, so
# GitHub's camo proxy can't fetch it and the image shows as broken. The ../ hops out of /pull/<n>/.)
EMBED="../blob/$BR/$DEST?raw=true"
```
(API-only verification → no image; skip 4a and put the fenced request/response in the comment.)

> **`mktemp` gotcha:** `GIT_INDEX_FILE=$(mktemp)` creates an *empty* file that git rejects
> (`index file smaller than expected`). Use `$(mktemp -u)` (a name, not a file) so git writes a
> fresh index. Also: a brand-new orphan branch can take minutes to appear on
> `raw.githubusercontent.com`, and **never** appears for private repos — which is exactly why the
> embed uses the relative `?raw=true` blob link instead.

### 4b. Comment on the PR
```bash
gh pr comment --body "$(cat <<EOF
✅ **Verified:** <feature> (spec NNN) against the running dev app.

<one or two lines: what was exercised + observed result>

![demo]($EMBED)
EOF
)"
```
(`$EMBED` is the repo-relative `../blob/$BR/$DEST?raw=true` link — renders inline for private repos.)

### 4c. Linear (only if a ticket was resolved)
Confirm flags first (`linearis files --help`, `linearis attachments --help`,
`linearis issues discuss --help`), then upload + reference:
```bash
URL=$(linearis files upload "$IMG" --json | python3 -c "import sys,json;print(json.load(sys.stdin)['url'])")
linearis issues discuss <issue-id> --body "Verified <feature>: ![demo]($URL)"
```

## 5. Report

Summarize: verdict, method (curl/browser), artifact type, the **raw image URL**, the **PR
comment URL**, and the **Linear link** (or "no ticket — skipped"). Note that the dev server
is left running (kill via the CLAUDE.md command when done).
