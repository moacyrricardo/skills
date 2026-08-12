---
description: Turn a half-formed UI/flow idea into a faithful, clickable HTML mock in the target product's own look — before you know the flow. Runs a preflight to decide its fidelity basis (clone the real status-quo page and apply only your change, or compose from the design system for a net-new surface), asking you when it can't tell. Declares that basis in a toggleable annotation overlay and emits one self-contained HTML+CSS+JS file. A facsimile of the product, not a document about it.
argument-hint: <what to mock — a change to an existing screen, or a new surface> [notes]
---

Build a clickable HTML **mockup** for: **$ARGUMENTS**.

A mockup exists to make a fuzzy UI idea concrete **before** the flow is decided — something to react
to, correct, and eventually turn into a spec. It **is** the product (a facsimile), not a document
*about* it. That distinction sets every rule here and is why this command shares nothing with the
`rich-html` document tools.

## The posture: a mock is the product, not a document

Because the mock stands in for the real UI, the conventions that make a *document* good actively
**corrupt** it. Hold the opposite line:

- **Match the product's *real* theme** — even if the product is light-only. Do **not** make the mock
  viewer-theme-aware (no `prefers-color-scheme` adaptation, no light/dark toggle): that would lie
  about what the product looks like. The mock looks like the product, full stop.
- **No document chrome** — no citations, no "how this was made" panel, no honesty footer as
  furniture. The only meta-layer allowed is the **annotation overlay** (§3), and it is
  **toggleable** and off by default so the bare mock reads as the product.
- **Fidelity beats legibility** — reproduce the product's actual look even where it's cramped,
  fixed-width, or dense. You are not optimizing for the reader; you are optimizing for *resemblance*.
- **Never a neutral fallback.** A neutral-styled mock is worthless. If you don't have the product's
  look, you *derive or infer* one (§1 ladder) — you never ship generic.

## 1. Preflight — decide the fidelity basis and the format BEFORE generating

Do this first, every time. **Governing rule for every choice below: infer it from the prompt and the
conversation first; only if you genuinely can't, ask — never silently decide for the user.** Run the
checks, then route.

**Check A — what look can I anchor to?** Climb this ladder and use the **highest rung available**:

1. **A formal design system** — tokens/theme files, a component library, documented pages. *(real fidelity)*
2. **Product signals** — no formal system, but real cues exist: an existing stylesheet, a brand
   colour in the logo/README, marketing copy, the app name/wordmark. Derive the palette and feel
   from these. *(inherited fidelity)*
3. **Domain archetype** — no signals, but the app's *nature* is known: infer an appropriate look —
   pet/animal → warm, rounded, playful; health/clinical → calm blues/greens, generous spacing;
   finance → dense, sober, data-forward; admin/back-office → utilitarian tables; ai/chat → minimal,
   conversational. *(inferred — a labeled placeholder, see §3)*
4. **Can't tell the domain** → **ask the user** (this is an interactive tool — asking is correct,
   not a failure).

**Check B — is there a status-quo page for the target?** Search the repo/product for the actual
screen(s) the request touches (routes, templates, components, screenshots).

**Route:**

| Intent (from the prompt) | Status-quo page found? | → Do |
| :--- | :--- | :--- |
| **Change** an existing surface | **yes** | **Clone the real page, apply ONLY the requested change** (§2). *Best case.* |
| **Change** an existing surface | **no / can't tell which** | **Ask** — "which page?" or "is this actually new?" (the *target* is ambiguous, not the intent) |
| **New** surface | n/a | **Compose from the anchor** found in Check A (§2, greenfield) |
| Can't tell **change vs new** | — | **Ask** |

**Check C — form factor.** Is the mock **mobile-only**, **desktop-only**, or **responsive**? Infer
from the request and the product (e.g. "the mobile app" / a phone flow → mobile; "the admin
dashboard" → desktop; a marketing page or "works on both" → responsive). If you can't tell, ask.

**Check D — one version or several?** Does the user want **variations** to compare (alternative
layouts, an A/B, different states)? Infer from the prompt ("show me two options" → several; a single
concrete ask → one). If several, choose the **packaging** the same way — infer, else ask:

- **Separate files** — one self-contained mock per version. Best when the versions diverge a lot, or
  each will be reviewed / handed off on its own.
- **Single file with a version selector** — all versions in one file with an unobtrusive switcher
  that swaps the page between them. Best for quick side-by-side comparison of close variants.

State, in one line, how you read the request before you generate: the **fidelity basis**, the
**status-quo target**, the **form factor**, and the **version plan** (count + packaging).

## 2. Build the mock

**Status-quo diff (the common case).** Start from the *actual* current page — reproduce its real
structure and styling as the baseline — then apply **only** the change the user asked for. The value
is that the change reads as a **true delta in real context**; don't redesign the surroundings, don't
"clean up" unrelated UI. Fidelity is *inherited*, not reconstructed. Reproduce **enough** of the real
surroundings to place the change in context — you may **stub or omit unrelated heavy components** (a
big chart, a data grid you're not touching, a complex widget) to keep the mock focused, but
**declare what you stubbed** in the overlay (§3), so the omission is honest rather than mistaken for
the real screen.

**Greenfield (net-new surface).** There's no page to clone, so compose the screen from the anchor
you found in Check A — real design system if you have one, else derived/inferred look. Reuse the
product's real components and spacing so the new surface looks native.

Make it **clickable enough to demonstrate the intended flow** — the states, transitions, and
interactions the idea hinges on — using small vanilla JS. It is a mock, **not a working app**: no
real data layer, no backend; fake the data and the happy path.

## 3. The annotation overlay — declare the basis and the inventions

The mock carries exactly one meta-layer: a **toggleable overlay** (a small fixed control, e.g. an
"ⓘ annotations" button), **off by default** so the clean mock reads as the product. When on, it shows:

- **Fidelity basis** — one honest line about where the look came from:
  *"based on `<real page/route>` + your change"* · *"composed from the design system"* ·
  *"derived from product signals (logo/README)"* · *"inferred <domain>-style — no design system found,
  confirm or point me at one."*
- **Invented components** — anything you drew that doesn't exist in the product yet, marked as
  invented (not passed off as real). Where you guessed a value (a colour, a label), say so.
- **Stubbed / omitted & substituted** — real parts of the screen you left out or simplified to focus
  the mock (§2), and any asset you substituted (a system font for the brand web font, a text wordmark
  for the logo). Keeps the mock from being mistaken for the full, exact screen.

This keeps the fidelity source **auditable at a glance** and preserves the honesty line the posture
demands without turning the mock into a document.

## 4. Emit the mock — self-contained, to the chosen form factor and version plan

Each `.html` must render identically as a local `file://`, an email attachment, or a Claude Artifact
(strict CSP — blocks every external request):

- **Inline all CSS and JS**; no CDN links or `fetch`/network. Keep it under ~16 MB.
- **Real fonts & assets — fidelity vs self-containment.** The product may use a **web font** (e.g.
  loaded from Google Fonts) or **image assets** (logo, icon set) you can't fetch at runtime. Resolve
  each one of two ways and **declare which** in the overlay: **embed it as a `data:` URI** when the
  look genuinely depends on it (the brand font, the logo), or **substitute the nearest system
  equivalent** (a matching system-font stack; an inline-SVG or styled-text stand-in for the wordmark)
  when a close match is good enough. Prefer inline SVG + system-font stacks by default; never leave a
  broken external reference.
- **Escape** any real text pulled from the product so it can't break layout.
- Remember the posture: the file's own styling is the *product's* theme, not a viewer-adaptive one.

**Form factor (Check C).** Build to the answer — a phone-width device frame for **mobile**, the
product's real desktop width for **desktop**, a genuinely reflowing layout for **responsive** (and
for responsive, verify it holds at both a narrow and a wide width).

**Versions (Check D).**

- *Separate files* → emit one self-contained file per version, named so the variant is obvious
  (e.g. `<slug>-v1-<label>.html`).
- *Single file + selector* → put every version in one file with a small switcher that swaps which
  one is shown.

**Presentation shell vs product surface.** The version switcher, the annotations toggle (§3), and any
device frame are **shell** controls — they live *outside* the product surface (think of the browser
frame around a screenshot) and are the *only* chrome allowed. They never bleed into the mock itself,
so the product surface still reads as the real thing. This is the one reconciliation of the "no
document chrome" posture: chrome *around* a facsimile is fine; chrome *in* it is not.

## 5. Deliver

Write the `.html` (or, for separate-file versions, all of them) to a sensible path (the user's
scratchpad unless they name a location) and hand it/them back with the file tools; offer to publish
as an Artifact for a shareable link. Say in one line what flow it demonstrates, plus the fidelity
basis, form factor, and version plan you used — and invite correction: a mock is made to be
re-skinned ("no, more like X") and iterated cheaply.

## Notes

- **Project-agnostic.** The design system, the status-quo pages, and the app's domain all come from
  the *target* project (its repo + `CLAUDE.md`) — nothing is hardcoded here.
- **Read-only except for the output file.** Inspecting the product to build the mock changes
  nothing; the command writes exactly one HTML file.
- **Interactive by design.** Unlike a background agent, this command *can and should* ask when the
  preflight is ambiguous — a wrong guess about intent or target wastes a whole mock.
- A finished mock is a natural seed for a **spec-workflow concern**'s design reference (its output
  can become the visual the concern points at). That hand-off is a soft, future integration — this
  command has no dependency on spec-workflow and works standalone.
