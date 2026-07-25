---
description: Build a ready spec end-to-end via the spec-workflow:spec-driver agent (background) — opens the PR on the todo→doing commit, pushes as-you-go, posts 3 evidence comments (tests, spec-eval, finish-branch), then captures a live GIF of the running feature via the test agent and posts it as a 4th comment. Stops at an open PR; never merges.
argument-hint: <spec-number> [extra notes]
---

Build spec **$1** autonomously **in the background** and leave it at an OPEN, evidence-backed PR.

This is an orchestration recipe, not new behavior: it wraps the existing `spec-workflow:spec-driver` agent in a
background `Workflow` and layers a fixed PR convention on top. It is **project-agnostic** — every
build / test / branch / version specific comes from the *target project's* `CLAUDE.md` and from
`spec-workflow:spec-driver`. Do not hardcode any project's toolchain here.

`$1` is the spec number. Any text after it in "$ARGUMENTS" is extra caller notes — pass it through
to the agent verbatim.

## Before launching (main thread, quick — do these yourself, not in the workflow)
1. Ensure this is a git repo, then land on a synced main: `git checkout main && git pull --ff-only`.
   (If the spec's own doc PR was just merged, this pulls it in.)
2. Locate the spec file `specs/$1-todo-*.md`. If it is missing, or already `doing`/`done`, **STOP**
   and tell the user — do not guess which spec they meant.
3. Delete any stale local branch left over from the spec's doc PR so the build recreates it cleanly.
4. While the workflow runs it drives **this same checkout** — don't run other git/build commands in
   it until it finishes (or have the agent use worktree isolation if the project prefers that).

## Part A — Build (background Workflow, one spec-workflow:spec-driver agent)
Call the `Workflow` tool with a single-phase script whose one `agent()` call sets
`agentType: 'spec-workflow:spec-driver'` and passes the prompt below (substitute the spec number; append the
caller notes). It runs in the background — report the task id, then stop and await the completion
notification. Do not stream progress. **When it completes and the PR is open, proceed to Part B**
(track it as a pending step so the long build doesn't make you forget it).

**Agent prompt to pass to spec-workflow:spec-driver:**

> Build ONE spec end-to-end: **spec $1** in this repo. Build ONLY this spec — do not drain the
> backlog or touch other specs. Finish to an OPEN PR and stop. **Never merge, never push to main.**
>
> **Environment:** Read the project's `CLAUDE.md` for the mandatory toolchain (e.g. a required
> JDK / `JAVA_HOME`), the build command, and how to run tests for a single module — use those
> exactly, do not assume. Read the spec file `specs/$1-*.md` **fully** before writing code.
>
> **Branch + PR FIRST (the very first commit is the status flip):**
> 1. Create the spec branch per the project's convention (a Linear-issue branch if one exists,
>    else the project's spec-branch pattern from `CLAUDE.md`).
> 2. First commit = `git mv specs/$1-todo-*.md specs/$1-doing-*.md` and flip the header
>    (`Status: doing`, `Branch:` = the branch). Subject: `spec-$1 Start build (todo→doing)`, with a
>    `Co-Authored-By:` trailer naming the model doing the work.
> 3. **Push the branch to origin immediately.**
> 4. **Open the GH PR now** (base main) so the PR's first commit IS the todo→doing flip. Capture the
>    PR number. End the PR body with the standard Claude Code footer.
>
> **Push discipline:** after EVERY commit below, immediately `git push`. Never batch commits.
>
> **Implement:** build the spec's decision as the smallest correct change, one logical commit per
> concern, each pushed as written. Follow the project's code style and architecture rules
> (`CLAUDE.md` / `ARCH.md`).
>
> **Tests → COMMENT #1:** add the tests the spec calls for, plus regressions for the exact
> bug/behavior. Run them with the project's test command; capture the report. Commit + push. Then
> post PR **COMMENT #1** titled `## Tests`: first list each NEW test by name with a one-line
> statement of what it proves, THEN the actual test-run report (pass/fail counts + summary).
>
> **Spec-eval → COMMENT #2:** evaluate the branch against the spec with the `/spec-workflow:spec-eval` discipline
> (read-only judgement): does it satisfy the Decision? gaps? in-scope fixes vs. items to defer to
> fresh specs? If the project's `CLAUDE.md` defines a version-bump policy, decide **explicitly**
> whether this change warrants a bump and recommend accordingly. Post PR **COMMENT #2** titled
> `## Spec eval`: the findings + a clear recommendation (SHIP as-is / apply these in-scope fixes /
> defer these). Then APPLY the in-scope fixes you recommended (including a fully-synced version bump
> if recommended), commit + push, and re-run tests if code changed.
>
> **Finish → COMMENT #3:** finish with the `/spec-workflow:finish-branch` discipline — the FINAL commit updates
> the spec (`git mv` doing→done, `Status: done`, keep the `Branch:` ref, add a short "how the
> implementation differed from the spec" note). Commit + push. Post PR **COMMENT #3** titled
> `## Finish-branch`: the finish output (final-commit summary, doing→done delta, how it differed).
>
> **Stop** at the open PR. Return a concise report: PR URL/number, the commit list, the test
> result, the spec-eval recommendation, and confirmation all THREE comments were posted.
>
> Extra caller notes (may be empty): $ARGUMENTS

## Part B — Live evidence (main thread, after Part A completes)
When the build workflow reports the OPEN PR, capture proof the feature actually **runs** (Part A's
tests are unit-level) and post it as **PR Comment #4** titled `## Live evidence`. This part is
deliberately **not** in the workflow: live capture needs a running app (and sometimes a real
browser), and the dev-server lifecycle / publish are most reliable driven from the main thread.

**Pick the evidence type from what the spec changes — do not assume a GIF:**
- **Visible UI change** → a **GIF** of the running flow showing the new/changed behavior.
- **Backend / API / logic change with an observable output** (endpoint response, status code,
  computed value, an emitted log line) → **capture that output directly**: a curl request/response
  or the relevant log excerpt. The fenced text IS the artifact.
- **No directly observable output** (pure refactor, or an opaque value like a token/hash format —
  e.g. spec 122) → exercise the flow that runs the changed code and show it works end-to-end as
  **regression proof**, and **label it as such**: the affected request returns clean / the page
  renders / the previously-failing path no longer errors — via API or a UI-flow GIF, whichever
  exercises the path.

Default to the most **direct** proof of the changed behavior; fall back to regression-of-the-flow
only when the change itself is unobservable.

**Prefer the project's `/spec-workflow:spec-test-live` skill** — it orchestrates this end to end (start the dev
server via `/live-verify:run-local-dev`, drive the `live-verify:test-flow-headless` agent, publish image artifacts to the
`verification-artifacts` branch). Invoke it for spec $1's feature against the **build branch**, tell
it which evidence type fits (per the heuristic above), and make sure the proof lands as a comment on
the **build PR**.

If `/spec-workflow:spec-test-live` is unavailable, do it manually — all app-specifics from the project's
`CLAUDE.md` "Dev server" + "UI evidence" sections:
1. Check out the build branch and build it with the project toolchain.
2. **Kill any stale dev-server instance first**, then start the dev server per the project's recipe
   and wait for its readiness signal.
3. Spawn the **`live-verify:test-flow-headless`** agent with the flow to exercise for spec $1 and the artifact
   type from the heuristic (`gif` for visual; a captured request/response or log excerpt for
   backend). It only *consumes* the running app — it never starts/stops it or publishes; you publish.
4. If the flow needs auth, **pre-stage the project's dev-login on the main thread** (e.g. the
   hash-swap recipe) and **ALWAYS restore it after** — a subagent crash must not strand mutated
   credentials.
5. Post **PR Comment #4** titled `## Live evidence` with a one-line note on WHAT it proves (feature
   behavior vs. regression of the affected flow), then the artifact:
   - **Image (GIF/PNG):** host it and embed the inline link by following the publish recipe in
     **`/spec-workflow:spec-test-live` §4** (§4a host on the `verification-artifacts` orphan branch, §4b comment) —
     that file is the single source of truth for the mechanics; don't restate them here.
   - **Text (API request/response or log excerpt):** paste it inline as a fenced block — no hosting needed.
6. Stop the dev server.

If live capture genuinely can't run here (no browser/display), fall back to an API/curl proof and
**say so** in the comment — never fake a pass, and never block the PR on capture tooling.

## Notes
- Requires the `spec-workflow:spec-driver` agent (shipped by this plugin) and the `Workflow` harness to be available.
- Part B additionally needs the project's dev-server + UI-evidence tooling (`/spec-workflow:spec-test-live`,
  `/live-verify:run-local-dev`, the `live-verify:test-flow-headless` agent) and, for authed flows, the project's dev-login
  recipe — restore any mutated credential afterward.
- The 3 comments deliberately mirror `/spec-workflow:spec-eval` and `/spec-workflow:finish-branch`; in a project without those
  skills, spec-workflow:spec-driver still applies the same discipline inline.
- Depends on `gh` being authenticated and the project having a GitHub remote.
