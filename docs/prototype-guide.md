# prototype — see the change before you spec it

A short guide to the `prototype` plugin and its one command, `mockup`. It turns a half-formed
UI/flow idea into a **faithful, clickable HTML mock in the target product's own look** — *before*
you know the flow, so you have something concrete to react to and eventually turn into a spec.

> 📖 **Prefer the visual version?** Open [`prototype-guide.html`](./prototype-guide.html) in a
> browser for the same guide with the preflight diagram and nicer formatting. This Markdown page is
> the quick, GitHub-readable companion.

```
idea → preflight → clone the real page + your change → a clickable mock
```

## Install

```
# 1 · register the marketplace (once)
/plugin marketplace add moacyrricardo/skills

# 2 · install
/plugin install prototype@moacyr-skills

# 3 · activate in the current session
/reload-plugins
```

Once installed the command is namespaced: `/prototype:mockup`.

## The idea: a mock *is* the product

The thing that makes `mockup` different from a document generator is that a mock **is** the product
— a facsimile of the real UI — not a document *about* it. That single distinction sets every rule:

- **Matches the product's *real* theme** — even if the product is light-only. It is *not*
  viewer-theme-aware; a light/dark toggle would lie about what the product looks like.
- **No document chrome** — no citations, no "how this was made" footer. The only meta-layer is a
  **toggleable annotation overlay** (off by default), so the bare mock reads as the product.
- **Fidelity beats legibility** — it reproduces the product's actual look, even where that's dense
  or cramped. It is optimizing for *resemblance*, not readability.
- **Never a neutral fallback** — a generic-looking mock is worthless (see the ladder below).

This is exactly the opposite posture from the `rich-html` document tools — which is why `mockup`
lives in its own plugin and shares nothing with them.

## The preflight — four things decided before a pixel is drawn

`mockup` never guesses silently. The governing rule for every choice below is **infer from the
prompt and the conversation first; only if you genuinely can't, ask — never decide for you.**

| Check | Decides |
| :---- | :------ |
| **A · fidelity basis** | Clone the real **status-quo page** and apply *only* your change (the common case), or compose from the design system for a net-new surface. |
| **B · status-quo target** | Which real page the change touches. Ambiguous? It asks. |
| **C · form factor** | mobile-only · desktop-only · responsive. |
| **D · versions** | one, or several to compare — and if several, **separate files** vs a **single file with a version selector**. |

## The fidelity ladder — where the look comes from

When there's no formal design system, `mockup` climbs a ladder rather than falling back to
something generic. Neutral is never an option:

```
design system  →  product signals  →  domain archetype (labeled)  →  ask
   (real)         (logo, brand         (pet → playful; finance →
                   color, existing       sober; clinical → calm
                   CSS, app name)        blues… — a placeholder)
```

Whatever rung it lands on, the mock **declares its basis** in the annotation overlay, so you always
know whether you're looking at real fidelity or an inferred placeholder.

## How to use it

### 1. Describe the change

Point `mockup` at what you want to see — a change to an existing screen, or a brand-new surface.
The more concrete ("add a filter bar to the transactions list"), the less it has to ask.

```
/prototype:mockup add quick-filter chips to the transactions list
```

### 2. Let the preflight resolve

It locates the design system and the real page, reads your intent, and settles form factor and
versions — asking you only where it genuinely can't infer. It states its read in one line before
generating.

### 3. Clone + apply only the delta

For a change, it reproduces the real page as the baseline and applies **only** what you asked —
stubbing unrelated heavy components (a big chart, an untouched grid) to keep the delta in focus, and
declaring what it stubbed. The change reads as a **true delta in real context**.

### 4. Open it, react, iterate

You get one self-contained HTML file — clickable enough to demonstrate the flow (not a working app).
Flip on the annotation overlay to see the fidelity basis and anything invented. A mock is made to be
re-skinned ("no, more like X") cheaply.

## What you get

- **One self-contained file** — inline CSS/JS, no network, `data:` URIs — so it opens as a local
  file, an email attachment, or a Claude Artifact.
- **A toggleable annotation overlay** declaring the fidelity basis, any invented components, and
  anything stubbed or substituted.
- **Clickable enough** to demonstrate the states and transitions the idea hinges on.

**Project-agnostic.** The design system, the status-quo pages, and the app's domain all come from
the *target* project (its repo + `CLAUDE.md`) — nothing is hard-coded. A finished mock is a natural
seed for a spec-workflow **concern**'s design reference, but `prototype` has no dependency on it and
works standalone.

---

Part of [moacyrricardo/skills](https://github.com/moacyrricardo/skills) · see the
[README](../README.md) for the full plugin reference.
