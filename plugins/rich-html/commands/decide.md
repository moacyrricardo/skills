---
description: Aggregate the decisions currently waiting on you — from concerns, PR review threads, issue questions, TODOs, autopilot escalations — into one rich, interactive HTML surface where each decision expands in place to its full context, you pick an option per decision, and the page emits a single prompt to hand back to an agent to execute them all. Not chat Q&A — a durable surface where a decision is made against real evidence.
argument-hint: <where to look for pending decisions> [scope notes]
---

Build an interactive **decision surface** for the decisions pending in **$ARGUMENTS**.

`decide` is not "ask the user a question." It gathers the **open decisions scattered across your
work** into one place, presents each with the *real evidence* needed to choose, records your
choices, and emits **one prompt** that an agent can run to carry them out. The evidence-backed card
**is** the value — it is what separates deciding-against-evidence from an LLM interrogating you
line by line in chat.

**Format contract:** the *how* of the file — self-containment / Artifact CSP, theme-awareness, and
the progressive-disclosure (expandable-context) pattern — comes from the **`html-doc`** skill. Read
`${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first and conform to it. This command owns the
decision model and the prompt it emits.

Terms: a **decision** is one card; its **question** is the card's one-line title; an **option** is
one selectable answer; the **choice** is the option picked.

## 1. Collect the pending decisions

Sweep the sources in scope for things genuinely **waiting on a human choice** — not facts, not
work-in-progress, but forks where someone must pick:

- **Concerns / design docs** — a `specs/` concern's `## Options` / `## Hypotheses`, or a
  `#comment:` asking for a call.
- **PR review threads** — unresolved questions, "should we A or B?", requested-changes forks.
- **Issue tracker** — issues flagged "Needs decision", open questions on a thread (via the
  project's tracker CLI if `CLAUDE.md` documents one).
- **Autopilot / automation escalations** — blockers a background agent parked for a human (draft-PR
  "🚧 Blocked — needs decision" sections, escalation comments).
- **In-code** — `TODO(decide):` / `FIXME` markers that encode an unmade choice.

With no argument, sweep the current repo: its `specs/`, the current branch's PR, and the tracker
per `CLAUDE.md`. One scoping question is allowed; a decision itself is never asked in chat.

Skip anything that is a task rather than a decision. If nothing genuine is pending, write no file
and say so plainly in chat — never manufacture choices.

## 2. Model each decision

Decisions are `D1…Dn` in page order; options are lettered `A…D`, each with a label of six words or
fewer. Conflicts, the prompt and every cross-reference name an option as `D2·B`. For each decision:

- **The question** — one line, the fork stated so it can be answered by picking.
- **What** — one paragraph: exactly what is being decided, and what is already decided around it
  that bounds it. (Givens for the whole set go in the lede, not on every card.)
- **Why** — one paragraph: what changed or broke to make this a fork, and what waiting costs.
  What and Why are always visible: the reader must be able to choose from the card alone; the
  evidence is proof, not prerequisite.
- **A today visual (nudged, not forced)** — one figure under Why showing the status quo and *where
  the fork sits in it*: mark the open spot (a dashed arrow, a highlighted "?" box), mark decided /
  broken / open with `--good` / `--bad` / `--warn` plus a label (never color alone), and say which is
  which in a `<figcaption>`. The test: *would the reader understand the fork faster with the picture
  than with one more sentence?* If not, omit it — an empty or decorative figure is worse than none.
  Only when the test passes does the subject suggest the form (how each is drawn: `html-doc` §4).
  These are illustrations, not a closed list:

  | The fork is about… | Form |
  | :-- | :-- |
  | a screen region — "bulk-action bar above the table, or floating?" | UI sketch (divs+CSS) |
  | stored shape — "one `status` column, or a `status_history` table?" | schema: entities, fields, relations |
  | an actor's path — "confirm the address before or after payment?" | use-case flow strip |
  | who calls whom — "`billing` calls `ledger` directly, or via the queue?" | service or sequence diagram |
  | code structure — "split `OrderService`, or extract a `Pricing` port?" | class / module diagram |
  | durations or dates — "cut over before the freeze, or after it?" | timeline — a chart, so load `dataviz` |
  | a name, a policy, a threshold — "`workspace` or `team`?", "keep logs 30 or 90 days?" | **none — prose** |

  One form per card: when a card mixes subjects, draw the aspect in which the options differ most.
  Card figures use only drawable forms (diagram, chart, UI sketch); screenshots and code stay in the
  evidence. Label nodes with the source's own identifiers (`ledger.post()`, not "Service B") — a
  figure you could paste unchanged into another project's card is decoration.
- **Options** — 2–4 real alternatives. A yes/no fork is two (do it / leave as is); if the source
  lists more than four, keep the four the evidence best supports and name the rest in the evidence.
  Each option has:
  - **How** — one paragraph on what would be done: what gets built or changed, where, in what
    order. Only how — it reads the same whether or not you like the option:
    - Leaky: "Move the check into the service. This is the cleanest option and avoids
      duplication, though the admin override gets riskier."
    - Pure: "`ExportService.run()` gains the size check before it enqueues; `ExportController`
      drops its own; `AdminExportController` passes `force=true`. One migration adds
      `exports.max_rows`."

    "Cleanest", "avoids" and "riskier" are trade-offs; they belong below.
  - **An outcome figure, when the card has a today visual** — how things look if this option is
    chosen, in the **same form and baseline** as the today visual with only the delta highlighted,
    plus a one-line good / warn / bad note. All options get one or none do; a "leave as is" option
    says "same as today" instead of repeating the figure. With no status quo to draw (greenfield),
    options may still carry figures, in one shared form.
  - **Trade-offs** — its gains, its losses, and one "wrong call when…" sentence, in a collapsed
    `<details>` whose summary shows the counts ("3 gains · 2 losses").
  - **Tags**, always visible — `recommended` with one visible sentence ("Recommended because …"),
    at most one per card and only when the evidence supports it; plus up to three free-form risk
    tags of three words or fewer (`cross-repo`, `needs migration`), each backed by a line under
    its losses.
  - **Conflicts** with options on other cards, when they exist — see below.
- **Defer — leave open** — a fixed last choice on every card. It is not counted in the 2–4, never
  recommended, never in conflict, and is emitted as "deferred — do not act".
- **Evidence** — the real source behind the fork, in a collapsed `<details>`: the concern's options
  section, the PR thread, the issue excerpt, the code around the marker (with the path in, per
  `html-doc`). Wire it to the actual source, not a paraphrase, and escape interpolated text; the
  figures above are drawn from it.
- **Source** — where it came from: PR#, issue id, concern number, or `path:line`.

### Conflicts between decisions

When an option on one card can't coexist with an option on another, declare it on **both** options,
as text that is always in the markup:

- **Hard** — an agent could not execute both: they give the same file, field, resource or contract
  incompatible states, or one removes what the other needs. Hard conflicts disable: once D2·A is
  picked, D1·C shows "Unavailable — contradicts D2·A: it removes the `sessions` table this option
  reads." Un-picking D2·A re-enables it. "Requires D2·B" is written as hard conflicts with D2's
  other options.
- **Soft** — both can be executed, but one erodes the other's gain or adds rework. Soft conflicts
  never disable; they show an inline warning on both options when both are picked ("Tension with
  D1·A: two cache layers to invalidate").

Declare a conflict only with evidence you can cite; when unsure, call it soft. List every pair once
in the lede ("Conflicts: D1·C ✕ D2·A (hard) · D3·B ~ D1·A (soft)"). Before writing the file, check
that no two `recommended` options hard-conflict (if the evidence recommends both, tag neither and
say so on both cards), and that no set of picks can leave a card with only Defer selectable (if one
could, those decisions are really one — merge them into one card).

## 3. Build the surface (per `html-doc`)

One self-contained interactive HTML file:

- A **sticky header**: the title, an "n / N decided" counter, **Accept all recommended**,
  **Compare trade-offs**, **Reset**, and the theme toggle.
- A **lede**: the sources, what is already decided or out of scope for the whole set, the conflict
  list, and a jump list to the cards.
- **Order**: with several sources, group the cards under an `<h2>` per source; with one source, use
  its own sections (severity, spec headings). Within a group, a decision others hinge on comes
  first. Under about eight cards, skip the group headings.
- **Each card** — structure only, not content; class names, styling and option count are yours:

  ```html
  <article class="card" id="D2">
    <h3>D2 · <question></h3> <span class="status">undecided</span>
    <p><!-- WHAT --></p>
    <p><!-- WHY --></p>
    <figure><!-- OPTIONAL today visual — nothing to draw: omit the element -->
      <figcaption>…</figcaption></figure>
    <div class="opt" data-opt="D2·A" data-conflicts="D1·C:hard">
      <label><input type="radio" name="D2" value="A"> A · <short name></label>
      <span class="tag">recommended</span> <span class="why">Recommended because …</span>
      <p class="conflict">Contradicts <a href="#D1">D1·C</a> (hard): <reason>.</p>
      <p><!-- HOW only --></p>
      <figure><!-- outcome figure, only if the card has a today visual --></figure>
      <details class="tradeoffs"><summary>3 gains · 2 losses</summary>
        <!-- gains · losses · wrong call when… --></details>
    </div>
    <!-- …further options, then "Defer — leave open" -->
    <details class="evidence"><summary>Evidence</summary><!-- real source excerpts --></details>
  </article>
  ```

  Only trade-offs and evidence are collapsed; everything else on the card is visible.
- **Picking** — each option is a `<label>` wrapping a native radio, and only that label selects:
  clicks inside `<details>`, on links or on figures don't. Clicking the chosen option again clears
  it. A pick opens that option's trade-offs and closes its siblings' (unless Compare is on), marks
  the card "decided", and applies its conflicts.
- **A disabled option** stays fully readable at full contrast; only its radio is disabled
  (`aria-disabled`), and its reason line stays visible.
- **Compare trade-offs** opens or closes every option's trade-offs on the page; it never touches the
  evidence.
- **Accept all recommended** fills only undecided cards that have a recommended option and never
  overwrites a pick. It skips a recommended option the reader's picks have disabled and says so
  inline ("1 skipped — conflicts with D1·B"). Hide it when no card has a recommendation.
- **Persistence** — save picks to `localStorage` (inside try/catch, keyed by the file) and restore
  them on load; **Reset** clears them.
- Without JS every card still reads in full — what, why, figures, each option's how, trade-offs and
  conflicts; JS adds selection, disabling and the prompt.
- **Guardrail:** from one decision up, the surface always renders as a rich document. It must
  **never** degrade into a chat Q&A fallback.

## 4. Emit one prompt — the payload

At the bottom of the page a read-only `<pre>` shows the prompt, rebuilt on every pick; **Copy
prompt** copies it, and the visible `<pre>` is the fallback when the clipboard is blocked. Copy is
enabled once every card has a choice — Defer counts. The prompt is plain text with no markup and
must stand on its own. Its format is fixed; only the wording comes from the picks:

```
Execute the following decisions in <repo> (<branch>). The trade-offs below were seen and accepted
when choosing — do not reopen them; stop and ask only if a listed loss proves worse than written.

D1. <question> — source: <PR# | issue id | concern | path:line>
    Chosen: A · <short name> — <how, in one sentence>
    Gains: <…>; <…>
    Accepted losses: <…>; <…>
    Revisit if: <wrong call when…>
D2. <question> — source: <ref>
    Deferred — do not act.
Tensions accepted: D1·A ~ D3·C — <reason>.
Record each decision at its source (resolve the thread, update the concern).
```

## 5. Deliver

Write the file to `<scratchpad>/decide-<YYYY-MM-DD>-<slug>.html` unless the user names a path, and
hand it back with the file tools; offer to publish as an Artifact for a shareable link. Tell the
user the flow in one line: open it → read the cards → pick → **Copy prompt** → paste back here (or
into a fresh session) to execute.

## Notes

- **Building the surface is read-only** — collecting decisions and rendering the file changes
  nothing. Execution happens only when the user runs the emitted prompt, deliberately, later.
- Project-agnostic: tracker/build specifics come from the target project's `CLAUDE.md`.
- Complements `report` (facts *about* things) — `decide` is for the forks *between* things. If a
  source turns out to hold reportable state rather than a pending choice, point at `report`.
