---
name: html-doc
description: The shared authoring contract for the rich-html plugin's document tools (report and decide) — how to build a single self-contained, theme-aware, interactive HTML file that renders anywhere (browser or Claude Artifact under a strict CSP) and uses progressive disclosure / expandable context to keep a dense document readable. Load this when generating a rich-html report or decision surface so the two feel like one system. This is the single source of truth those commands build on — prefer it over any inline restatement.
---

# Authoring a self-contained interactive HTML document

`report` and `decide` both emit **one HTML file** that a person opens and interacts with. This
skill is the format contract they share so their output feels like one system. It is *not* about
what to put in the document (each command owns its content) — only *how the file is built*.

The three non-obvious constraints below are the whole point of the skill; a capable model will get
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

- Default to the OS preference via `@media (prefers-color-scheme: dark)`.
- Let a toggle **win in both directions** by stamping `data-theme` on the root and styling
  `:root[data-theme="dark"]` / `:root[data-theme="light"]` overrides. Canonical toggle:
  ```html
  <button class="themebtn" onclick="var r=document.documentElement;
    r.setAttribute('data-theme', r.getAttribute('data-theme')==='dark'?'light':'dark')">◐ theme</button>
  ```
- Drive all color from a small set of CSS custom properties (`--bg`, `--fg`, `--muted`,
  `--accent`, `--border`, `--card`) redefined per theme — never hard-code a hex in a rule.
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

Both commands start from the same shape — a fixed header with the theme toggle, then content:

```html
<!doctype html><html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>…</title>
<style>
  :root{ --bg:#fff; --fg:#1a1a1a; --muted:#666; --accent:#2563eb; --border:#e5e7eb; --card:#f8f9fa; }
  @media (prefers-color-scheme:dark){ :root{ --bg:#0f1115; --fg:#e8e8e8; --muted:#9aa0a6; --border:#2a2d34; --card:#171a20; } }
  :root[data-theme="dark"]{ --bg:#0f1115; --fg:#e8e8e8; --muted:#9aa0a6; --border:#2a2d34; --card:#171a20; }
  :root[data-theme="light"]{ --bg:#fff; --fg:#1a1a1a; --muted:#666; --border:#e5e7eb; --card:#f8f9fa; }
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
  the user wants a shareable link — Artifacts render this format natively). Both commands say where
  their output lands.
- One document = one file. If a command produces a companion machine payload (e.g. `decide`'s
  emitted prompt), that is separate from this HTML, not embedded chrome inside it.
