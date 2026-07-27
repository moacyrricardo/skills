# skills

My personal [Claude Code](https://code.claude.com/docs) plugins — custom skills, agents, and
commands I use every day, published as an installable marketplace.

This repo is itself a **plugin marketplace** (`moacyr-skills`). Each plugin lives under
[`plugins/`](./plugins) and is independently installable.

> 📖 **New here?** The [Spec Workflow guide](./docs/spec-workflow-guide.md) walks through going
> from a rough idea to an open PR ([rich HTML version](./docs/spec-workflow-guide.html)).

## Install

```shell
# 1. Register this repo as a marketplace (once)
/plugin marketplace add moacyrricardo/skills

# 2. Install a plugin from it
/plugin install spec-workflow@moacyr-skills   # pulls live-verify automatically (it depends on it)
# or grab the verification primitives on their own:
/plugin install live-verify@moacyr-skills

# 3. Activate in the current session
/reload-plugins
```

Update later with `/plugin marketplace update moacyr-skills`.

Prefer to try before installing? Clone the repo and point Claude Code at a plugin directly:

```shell
claude --plugin-dir ./plugins/spec-workflow --plugin-dir ./plugins/live-verify
```

> Plugin commands and skills are **namespaced** by the plugin, so `/build` becomes
> `/spec-workflow:build` once installed, and `run-local-dev` becomes `/live-verify:run-local-dev`.
> Agents are referenced by scoped name, e.g. `spec-workflow:autopilot`, `live-verify:test-flow-headless`.

## Plugins

### `spec-workflow` — spec-driven development

A workflow for turning decisions into shipped code through a durable, numbered catalog.
The unit of work is a **spec** (a decision, ready to build) or a **concern** (exploratory);
each lives as `specs/NNN-status-slug.md` and moves `todo → doing → done` by being renamed.
Depends on **`live-verify`** (below) for the live-app proof step.

**Commands**

| Command | What it does |
| :------ | :----------- |
| `new` | Create a new spec/concern in the `specs/` catalog (always on `main`). |
| `start` | Find/create a Linear issue, mark the spec `doing`, cut the feature branch. |
| `build` | Implement a spec end-to-end — code, tests — then hand off to `finish`. |
| `eval` | Read-only, thorough evaluation of the branch against its spec. |
| `finish` | Update the spec (status, branch, summary) and create the final commit. |
| `ship` | Drive a ready spec to an open PR via the `autopilot` agent, with evidence comments. |
| `test-live` | Verify a shipped feature against the running app and attach the proof to the PR/Linear (orchestrates `live-verify`). |
| `plan` | Produce a planning table from the catalog. |
| `setup` | Preemptively configure the workflow for a project: infer what it can from the codebase, ask for the rest, write it to `CLAUDE.md`. |
| `redteam` | Adversarially stress-test a `todo`/`doing` spec for gaps and inconsistencies. Report-only, cites spec text, never fabricates or acts. |
| `audit` | Check the catalog's structural integrity — CATALOG↔files, number collisions, cross-reference graph, and `NNN-assets/` link integrity. Report + fix proposal (applies mechanical fixes on confirm). |

**Agent**

| Agent | What it does |
| :---- | :----------- |
| `autopilot` | Autonomously drains the `todo` backlog, building each spec to an open PR. Never merges. |

**Skill**

| Skill | What it does |
| :---- | :----------- |
| `spec-conventions` | The shared model the commands and `autopilot` build on: concern vs spec, `NNN-status-slug` naming, the `todo→doing→done` lifecycle, the per-status section templates, and the `specs/` catalog + `CATALOG.md` index. Single source of truth — the commands reference it rather than restating it. |

### `live-verify` — prove a feature works against a running app

Reusable primitives with **no spec/tracker coupling** — usable on any project. `spec-workflow`
builds on these, but you can install `live-verify` alone to verify a feature and get a gif/screenshot.
It captures evidence; it never publishes — the caller does.

| Component | Type | What it does |
| :-------- | :--- | :----------- |
| `run-local-dev` | command | Start/confirm the project's local dev server (generic; reads `CLAUDE.md`). |
| `test-flow-headless` | agent | Exercises a feature against a running app (Firefox via geckodriver, or curl) and returns a verdict plus a screenshot/gif/text artifact. |

### `utils` — general-purpose helpers

A catch-all/playground for standalone tools that don't belong to a specific workflow. Grows as
I add small helpers; nothing here depends on the other plugins.

| Command | What it does |
| :------ | :----------- |
| `stacked-merge` | Squash-merge a PR or a chain of stacked PRs into `main`, handling the `--onto` rebase bookkeeping between each merge. Presents the plan for confirmation before executing. |

**Conventions this assumes.** These are extracted from my own setup, so they lean on a few
project conventions you can adapt. Nothing project-specific is hardcoded in the commands —
per-project values are read from the target repo's `CLAUDE.md` (the file Claude Code already
auto-loads). A minimal block:

```markdown
## Spec workflow
Linear team: ABC          # your Linear team key — start reads this, or asks and appends it

## Dev server
Start: <command>          # run-local-dev / test-flow-headless read this
Port:  <port>
Ready: <readiness signal>
```

- A `specs/` directory used as a numbered decision catalog (`NNN-status-slug.md`).
- [Linear](https://linear.app) for issue tracking (`start`, `ship`), keyed off the
  `Linear team:` line above. Swap in your own tracker or drop these steps if you don't use it.
- A `## Dev server` section in the project's `CLAUDE.md` that `run-local-dev` reads.
- GitHub + the `gh` CLI for PRs.

**One plugin caveat.** Claude Code ignores the `permissionMode` field on plugin-shipped agents,
so `autopilot` will prompt for edits rather than auto-accepting them. To restore the
hands-off behavior, add a `permissions.allow` rule in your `settings.json`, or copy
`agents/autopilot.md` into your own `~/.claude/agents/` (where `permissionMode` is honored).

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (this repo)
├── plugins/
│   ├── spec-workflow/            # depends on live-verify
│   │   ├── .claude-plugin/plugin.json
│   │   ├── commands/             # 11 slash commands
│   │   └── agents/               # autopilot
│   ├── live-verify/             # standalone verification primitives
│   │   ├── .claude-plugin/plugin.json
│   │   ├── commands/             # run-local-dev
│   │   └── agents/               # test-flow-headless
│   └── utils/                   # general-purpose helpers (playground)
│       ├── .claude-plugin/plugin.json
│       └── commands/             # stacked-merge
└── README.md
```

New plugins get their own directory under `plugins/` and an entry in `marketplace.json`.
A plugin declares a same-marketplace dependency via the `dependencies` array in its `plugin.json`
(e.g. `spec-workflow` lists `"live-verify"`), so installing it pulls the dependency automatically.

## License

MIT
