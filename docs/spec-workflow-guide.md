# Spec Workflow — from idea to an open PR

A short guide to going from a rough idea to reviewed, evidence-backed pull requests with these
plugins. You frame the idea, split it into buildable specs, and hand each one to an agent that
implements it, tests it, proves it runs, and stops at a PR for you to review.

> 📖 **Prefer the visual version?** Open [`spec-workflow-guide.html`](./spec-workflow-guide.html)
> in a browser (or via GitHub Pages, if enabled) for the same guide with the pipeline diagram and
> nicer formatting. This Markdown page is the quick, GitHub-readable companion.

```
idea → concern → specs → spec-ship → open PRs
```

## Install

The repo is a plugin marketplace. Installing `spec-workflow` pulls in `live-verify` automatically
(it depends on it).

```
# 1 · register the marketplace (once)
/plugin marketplace add moacyrricardo/skills

# 2 · install — spec-workflow pulls live-verify with it
/plugin install spec-workflow@moacyr-skills

# 3 · activate in the current session
/reload-plugins
```

Once installed, commands are namespaced by their plugin: `/spec-workflow:new-spec`,
`/spec-workflow:spec-ship`, `/utils:stacked-merge`, and so on.

## The model

Every decision lives in a `specs/` directory — the catalog. A file is named `NNN-status-slug.md`,
and **its status is its filename**. The whole plugin set shares this model (the `spec-conventions`
skill), so the commands never disagree about what a spec is.

- **Concern** *(umbrella)* — exploratory. Frames a problem, lays out options, keeps them open.
  **Not a decision yet.** One concern usually spawns several specs.
- **Spec** *(buildable)* — a decision made, detailed and prescriptive. Each spec is one bounded,
  shippable slice.

A spec advances by being **renamed** (each step its own commit):

```
todo  →  doing  →  done
```

## Idea → PR, in four moves

### 1. Frame the idea as a concern

Start wide. Run `new-spec` and choose **concern** — capture the problem, the options you're
weighing, and the open questions. You're not committing to an approach yet.

```
/spec-workflow:new-spec   → concern
```

### 2. Split it into a few specs

As the decisions firm up, break the concern into bounded, buildable slices — one **spec** each,
via `new-spec`. Mark the concern resolved with a note pointing at the specs that settled it. You
now have a small stack of `todo` specs.

```
/spec-workflow:new-spec   → spec   (repeat)
```

### 3. Ship each spec

Hand a spec to `spec-ship`. An agent builds it in the background: it flips `todo → doing` as the
first commit (so the PR opens on that flip), implements the change, and leaves an **open** PR —
stacked when specs depend on each other. It **never merges**. Run it once per spec.

```
/spec-workflow:spec-ship 003
```

### 4. Review, then merge

Each PR arrives with its evidence attached (below). You review and merge on GitHub. When specs
were stacked, `utils`' `stacked-merge` squash-merges the chain in order and handles the rebase
bookkeeping between them.

```
/utils:stacked-merge 3 5 6
```

## What `spec-ship` leaves behind

The point isn't speed — it's that each PR carries proof, so review is fast and grounded:

| Comment | What |
| :------ | :--- |
| **1 · Tests** | New tests named and explained, plus the actual pass/fail run. |
| **2 · Spec-eval** | A read-only judgement of the branch against the spec — gaps, in-scope fixes, deferrals. |
| **3 · Finish-branch** | The final commit that moves the spec `done` and records how the build differed from the plan. |
| **4 · Live evidence** | Proof it runs — a GIF of the flow, or a captured request/response — via `live-verify`. |

## The three plugins

| Plugin | Depends on | What |
| :----- | :--------- | :--- |
| **spec-workflow** | live-verify | The catalog lifecycle: `new-spec`, `spec-to-linear`, `build-spec`, `spec-eval`, `finish-branch`, `spec-ship`, `specs-table`, the `spec-driver` agent, and the `spec-conventions` skill. |
| **live-verify** | — | Prove a feature works against a running app: start the dev server, then drive it (browser or `curl`) to capture a screenshot / GIF / text verdict. No spec/tracker coupling. |
| **utils** | — | General-purpose helpers. Currently `stacked-merge` — squash-merge a PR or a chain of stacked PRs into `main`, rebasing between each. |

**Tracker-agnostic.** Nothing project-specific is hard-coded. Per-project values — your issue
tracker's team key, the dev-server command — are read from the target repo's `CLAUDE.md`. No
tracker? The Linear steps simply drop out.

---

Part of [moacyrricardo/skills](https://github.com/moacyrricardo/skills) · see the
[README](../README.md) for the full plugin reference.
