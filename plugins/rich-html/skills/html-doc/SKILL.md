---
name: html-doc
description: The shared authoring contract for the rich-html plugin's document tools (report, decide, and branch-review) — how to build a single self-contained, theme-aware, interactive HTML file that renders anywhere (browser or Claude Artifact under a strict CSP) and uses progressive disclosure / expandable context to keep a dense document readable. Load this when generating a rich-html report, decision surface, or branch review so they feel like one system. This is the single source of truth those commands build on — prefer it over any inline restatement.
---

# Authoring a self-contained interactive HTML document

`report`, `decide`, and `branch-review` each emit **one HTML file** that a person opens and
interacts with. This skill is the format contract they share so their output feels like one system.
It is *not* about
what to put in the document (each command owns its content) — only *how the file is built*.

The four non-obvious constraints below are the whole point of the skill; a capable model will get
the prose right on its own but will drift on these unless they are written down once.

## 1. Self-contained — one file, no network

The file must render identically opened as a local `file://`, emailed as an attachment, or
published as a Claude **Artifact** (which enforces a strict Content-Security-Policy that blocks
*every* external request). Therefore:

- **Inline everything.** All CSS in one `<style>`; all JS in one `<script>`; no CDN links, no
  external stylesheets, no web-font URLs, no `fetch`/`XHR`/WebSocket.
- **Embed assets as `data:` URIs** — images, icons, fonts. Prefer inline SVG and system-font
  stacks so there is nothing to embed.
- **No build step, no framework.** Vanilla HTML/CSS/JS. If you reach for a library, you have taken
  a wrong turn — the interactions here (toggle, expand, copy) are a few lines each.
- Keep the rendered file **under ~16 MB** (the Artifact ceiling; `data:` URIs count).

## 2. Theme-aware — respect the viewer's light/dark

The document is *about* something; it is not a product facsimile, so unlike a mock it **must**
adapt to the reader's environment and offer a manual override.

- Default to the OS preference via `@media (prefers-color-scheme: dark)`, scoped to
  `:root:not([data-theme="light"])` so a light choice beats a dark OS.
- Let a toggle **win in both directions** by stamping `data-theme` on the root and styling
  `:root[data-theme="dark"]` / `:root[data-theme="light"]` overrides. The toggle flips the
  *effective* theme — with no `data-theme` set yet, that is the OS preference — so its first click
  always switches. Canonical toggle:
  ```html
  <button class="themebtn" onclick="var r=document.documentElement, t=r.getAttribute('data-theme')
    || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    r.setAttribute('data-theme', t==='dark' ? 'light' : 'dark')">◐ theme</button>
  ```
- Drive all color from a small set of CSS custom properties (`--bg`, `--fg`, `--muted`,
  `--accent`, `--border`, `--card`, and `--good` / `--warn` / `--bad` for state) redefined per
  theme — never hard-code a hex in a rule. A state color always comes with a label or glyph.
- Write the document in the language of the user's request and set `lang` to match; quoted source
  text stays verbatim.
- **Optional project palette:** if the target project's `CLAUDE.md` points at a design system /
  palette, adopt its accent + surface colors for these variables; otherwise use a neutral,
  accessible default. This is a light decoration, not fidelity — a document reads as a document,
  not as the product (that posture belongs to `mockup`, and is deliberately the opposite here).

## 3. Progressive disclosure — dense but readable

A rich document earns its format by carrying real evidence **without drowning the reader**. Show
the conclusion; let the reader expand to the proof, in place.

- Use native `<details>/<summary>` for expandable context — it needs zero JS, is keyboard- and
  screen-reader-accessible for free, and prints/exports open-or-closed predictably.
- **Collapsed by default; expands in place** to the underlying evidence (a source excerpt, a
  changed-files list, a linked thread) — never navigate away, never a modal.
- Reserve custom JS for interactions `<details>` can't express (a copy-to-clipboard button, an
  "expand/collapse all", `decide`'s selection state). Keep it small and unobtrusive.
- The document must be **fully readable with JS disabled** — JS enhances, it never gates content.
  JS may open or close `<details>` and disable a control in response to an action, provided the
  content stays in the DOM, readable, and openable without JS.

## 4. Visual evidence — match evidence to its form

Every piece of evidence is **text plus, when it helps, a supporting artifact — never the artifact
alone.** The prose states the claim or the fork and carries the reader; a code snippet, chart,
diagram, or sketch sits *under* it — by default inside the expandable context, as substantiation the
reader can check. (A command may promote a visual into the always-visible body when the visual *is*
the fastest way to make the call — `decide`'s today visual and per-option outcome figures, and
`branch-review`'s summary chart, do.) So the question is never "text or visual" — it is whether a
supporting artifact earns its place beside the text. All three commands draw from the same small
vocabulary of supporting forms so their output stays one system; each command owns *when* a piece of
its content warrants one, this skill owns *how* each is built (and every form here is inline and
CSP-safe — SVG and divs inline, images as `data:` URIs):

- **Code → the real snippet, plus how execution reaches it.** Paste the actual lines (escaped, in
  `<pre>`) and add one line on the path in — the caller, the trigger — so the point rests on the
  code, not a paraphrase.
- **Flow / structure (state/page A → B, or a small system map) → a small inline-SVG diagram.**
  Labeled nodes and arrows, so the reader sees the shape instead of reading a sentence about it. The
  nodes are whatever the evidence is about: pages or states, data stages (sources → transforms →
  consumers), an actor's steps in order, services with labeled calls/events, modules with their
  dependencies, entities with their fields and relations (a schema), or classes with only the
  members that matter. For calls *over time*, draw a sequence: one lifeline per participant,
  messages top to bottom. Keep it a few nodes; it is a signpost, not a full architecture diagram.
  When a diagram's point is *timing* (a race, a retry, a state machine), it may step through its
  states with play / next controls and a caption per step — with reduced motion or JS off, all
  steps show at once.
- **Trend / distribution / comparison → a chart.** Inline SVG, no chart library. A duration or
  timeline bar against a budget is a chart too, so it follows this rule. When you draw one,
  **load the `dataviz` skill first** for palette, form choice, and light/dark-safe rendering rather
  than hand-rolling a bar — a report synthesizing history or logs is often *itself* a trend, so this
  is the form `report` reaches for most.
- **Many items × a few attributes → a table.** Real `<table>` with a header row, inside its own
  scroll container; when items vary on one number, it is a chart instead.
- **UI appearance → show it.** Embed the screenshot as a `data:` URI rather than describing how a
  screen looks. **Read-only caveat:** all three commands only inspect — use an image the source already
  has (a PR, an issue, a thread); never spin up the app just to capture one. With no existing image,
  fall back to a layout sketch or plain description.
- **UI region / layout → rough it in divs+CSS.** A few styled divs sketching the relevant part of a
  screen — one region, or each option's arrangement side by side so shapes compare at a glance. This
  one is **decide-flavored** — it serves *choosing between* layouts and showing where a fork sits in
  a screen, so `report` rarely needs it. And it is a **thumbnail, not a facsimile**: pixel fidelity
  to the product is `mockup`'s posture and deliberately not this skill's (see §2). Keep it schematic.

When a figure is repeated to compare states (today vs. each option), keep its nodes, order and
positions fixed and change only the delta:

```
today      [form] → [api] ⇢ ? ⇢ [db]
option A   [form] → [api] → [queue]* → [db]
option B   [form] → [api] → [db]*            (* = the only thing added or recolored)
```

If an option can't be drawn without rearranging the picture, the form is wrong for the comparison.

Every aid is **evidence, not decoration** — the text always stays; add a supporting artifact only
when it substantiates the claim faster than more prose would. And none of them gate content: because
the prose already carries the point, the document stays fully usable if an image fails to load or JS
is off.

## Accessibility & robustness (baseline, not optional)

- Semantic structure: one `<h1>`, real `<h2>`/`<section>` landmarks, real `<button>`s (not
  clickable `<div>`s), `<a>` for links.
- Meet WCAG AA contrast in **both** themes (check against the `--bg` of each).
- Responsive: relative units, flex/grid, `max-width:100%` on media. Wide content (tables, code,
  diagrams) scrolls inside its own `overflow-x:auto` container — the page body never scrolls
  sideways.
- Escape any interpolated source text (`<`, `>`, `&`) so a snippet from a PR or log can't break
  the layout or inject markup.

## The house skeleton

All three commands start from the same shape — a fixed header with the theme toggle, then content:

```html
<!doctype html><html lang="…"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>…</title>
<style>
  :root{ --bg:#fff; --fg:#1a1a1a; --muted:#666; --accent:#2563eb; --border:#e5e7eb; --card:#f8f9fa;
         --good:#15803d; --warn:#b45309; --bad:#b91c1c; }
  @media (prefers-color-scheme:dark){ :root:not([data-theme="light"]){ --bg:#0f1115; --fg:#e8e8e8; --muted:#9aa0a6; --border:#2a2d34; --card:#171a20;
         --good:#4ade80; --warn:#fbbf24; --bad:#f87171; } }
  :root[data-theme="dark"]{ --bg:#0f1115; --fg:#e8e8e8; --muted:#9aa0a6; --border:#2a2d34; --card:#171a20;
         --good:#4ade80; --warn:#fbbf24; --bad:#f87171; }
  :root[data-theme="light"]{ --bg:#fff; --fg:#1a1a1a; --muted:#666; --border:#e5e7eb; --card:#f8f9fa;
         --good:#15803d; --warn:#b45309; --bad:#b91c1c; }
  body{ margin:0; background:var(--bg); color:var(--fg); font:16px/1.6 system-ui,-apple-system,sans-serif; }
  /* … */
</style></head>
<body>
  <header>…<button class="themebtn" onclick="…">◐ theme</button></header>
  <main>…</main>
  <script>/* small, optional enhancements only */</script>
</body></html>
```

## Delivering the file

- Write the `.html` to disk and hand it back with the file tools (or publish as an Artifact when
  the user wants a shareable link — Artifacts render this format natively). Each command says where
  its output lands.
- One document = one file. A companion machine payload (e.g. `decide`'s emitted prompt) is plain
  text the page displays and copies; it never contains the document's markup.
