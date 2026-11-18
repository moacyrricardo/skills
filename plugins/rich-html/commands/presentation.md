---
description: Turn a story and its real sources into a self-contained HTML slide deck — one full-screen 16:9 slide at a time, keyboard / button / swipe navigation, speaker notes and a print view, each slide one claim with the evidence that proves it. Reads the sources, builds the deck, renders every slide to check it, and hands back one file. Read-only — it never changes or re-captures what it presents.
argument-hint: <the story, as beats> [audience / occasion / sources]
---

Build a rich HTML **presentation** of **$ARGUMENTS**.

A deck is an **argument in order**: each slide makes one claim and shows the thing that proves it.
It is not a report cut into pages. It reads its sources and **never mutates** them (no commits, no
issue edits, no pushes) and **never re-captures** the product — no starting the app, no harness,
no product screenshots. The output is one self-contained HTML file.

**Format contract:** self-containment, theme tokens, accessibility and the visual-evidence forms
come from the **`html-doc`** skill. Read `${CLAUDE_PLUGIN_ROOT}/skills/html-doc/SKILL.md` first
and conform to it. This command owns the deck shell, the slide vocabulary, the asset pipeline, the
render check, and the few places a deck departs from `html-doc` (§4).

## 1. Frame it — beats, effect, audience

- **Beats** — the user's ordered points. Keep their order; you choose how many slides each needs.
- **Effect** — what each beat should make the audience conclude *and feel*. Turn an adjective into a
  form you can count and show: "it feels bureaucratic" → every screen it takes today, drawn in
  order, with a tally; "it's slow" → the measured time against the budget. The layout carries the
  feeling; the content stays sourced. A playful frame is welcome when the slide says it is one —
  the real steps on a mock form, under a line like "the steps are real; only the letterhead is a
  joke". The frame may be invented; what's inside it may not.
- **Audience and occasion** — presented live (fewer words, notes matter) or sent to be read (a
  little more text per slide).

State your reading in one line and proceed. Ask **one** question only if a beat has no source.

## 2. Gather — real sources, looked at

- Draw from what exists: transcripts and messages, specs and docs, PRs and their attachments,
  existing captures (screenshots, GIFs), and the code for exact UI copy. The project's `CLAUDE.md`
  says where captures live and which design system applies.
- **Read every text you will quote, and open every image — and a few frames of every GIF — before
  writing about it.** A caption must describe what the asset shows, not what you expected; if it
  shows a neighbouring case, say so on the slide.
- **Quotes are verbatim**, with a locator (timestamp, line, URL). Note transcription doubt; never
  translate a quote silently. No source, no quote.
- Example data in a figure is labelled as example; a counted claim ("8 steps") lists what it counts.
- Nothing captured for a screen you need? Draw it (§3) and say on the slide that it is drawn. Draw
  every state the slide talks about — a menu *and* the dialog it opens — not just one of them.

## 3. Shape the slides — one claim, one form

- **Headline = the slide's claim**, as a sentence. A small overline names the beat.
  - Leaky: "Current workflow" — a topic label; the audience has to find the point.
  - Pure: "Filing a report takes eight steps and two trips back" — the point, checkable on the slide.
- **One figure per slide**, always visible. Body ≤ ~120 words; more than that is another slide.
- **A source line** at the foot of every slide that rests on a source.
- Pick the form from what the beat is *about* (how each is drawn: `html-doc` §4). Illustrations, not
  a closed list:

  | The beat is… | Slide form |
  | :-- | :-- |
  | what someone said | one hero quote + up to two supporting quotes, each with its locator; a voice beat may take more than one slide — keep the voice on the slides, not only in notes |
  | how a screen looks today | an existing capture with ≤ 3 numbered rings drawn around the exact controls, keyed to points beside it |
  | a path that is too long | the screens it passes through, drawn small and in order, each numbered and tagged with where it happens; mark revisits and the step where the missing context finally arrives; a tally of steps, clicks and screen changes — or the same steps in a labelled playful frame |
  | a screen or control with no capture | a UI miniature built from real markup and copy, showing every state the beat mentions, captioned as drawn |
  | a change | before / after on one baseline, only the delta highlighted |
  | it working | a captured GIF or still with numbered captions |
  | a trend or comparison | a chart — load `dataviz` |
  | the takeaway | one number or one sentence, plus what changes and what doesn't |

- One **cover** (title, a one-line lead, the beats as an agenda) and one **closing summary**. No
  section dividers, "questions?" or thank-you slides, and no decorative figures — every slide
  carries a claim or its evidence.
- Speaker notes are optional, per slide: what to say that the slide doesn't show.

## 4. Build the deck shell

**Start from the shell — don't write navigation yourself.** Copy
`${CLAUDE_PLUGIN_ROOT}/templates/deck-shell.html`, keep its `<style>` shell block and `<script>`
unchanged, and replace only what it marks `REPLACE`: the language, title, design tokens, button
labels and the slides. The shell is tested once, so every deck gets the same behaviour:

- **Navigation:** Previous / Next buttons, ← / →, PageUp / PageDown, Space, Home / End, swipe (the
  slide follows the finger), **F** full screen, **N** speaker notes, a progress bar, `#N` in the URL.
  Arrows keep working after a click on Next; Space on a focused button stays with the button.
- **Transitions:** `data-transition` on `<main class="stage">` sets the deck's default; the same
  attribute on one `<section class="slide">` overrides it for arriving at that slide:
  - `slide` (default) — the next slide comes in from the right, the previous from the left;
  - `slide-up` — the same, vertically; for a step deeper into the same beat;
  - `fade` — out, then in; for the cover, a change of beat, or a reveal;
  - `none` — instant.
  Keep one default and override sparingly. With reduced motion set, everything becomes a short fade.
- **Fit:** a 1280×720 stage scaled to the window. On a phone-sized screen (narrower than 700 px or
  shorter than 480 px) the shell stops scaling: the slide reflows to the screen width and scrolls,
  so give any multi-column layout a one-column fallback under `.stage.reflow`.
- **Without JS** slides stack in order with their notes visible; **print** uses the same stacked
  view, one slide per page, notes under their slide.

Each slide is a `<section class="slide">` with an overline (`.over`), its headline (`h1` on the
cover, `h2` elsewhere), one `<figure>`, a source line (`.src`) and optional notes
(`<aside class="notes">`). No click-to-advance — it fights clicks on figures and links.

- **Where a deck departs from `html-doc`:**
  - no `<details>` on a slide — the sequence is the disclosure; extra proof goes in the notes;
  - the slide's figure is promoted to the visible body (the `html-doc` §4 promotion clause);
  - captured or mimicked product UI keeps its native colours inside a framed figure — only the
    deck surface follows the theme;
  - a UI miniature may be faithful (`html-doc` asks for a schematic thumbnail) when it is built from
    real markup and copy, shows one region, and its caption says what is real and what is example.
- **Design system:** adopt the one `CLAUDE.md` points to (tokens, type, reserved colours); otherwise
  `html-doc`'s neutral default. If it has no dark palette, derive one that passes contrast in §6.

## 5. Assets

- Crop each capture to the region the slide talks about and downscale to its displayed width; JPEG
  for screenshots; keep GIFs short and small. Never crop through a row or a control, or away from
  the element a caption names. Budget ~5 MB for the file; report the final size.
- A GIF restarts when its slide is shown and has a replay button; its numbered captions follow the
  order of what it shows.
- **Keep base64 out of your context:** write the deck with placeholders (`@@shot-4@@`), then inline
  the assets with a small script that fails if any placeholder is left.
- `alt` on every image; callout numbers match the points beside the image.

## 6. Check it — render and look (don't skip)

- With a local headless browser (e.g. `chromium --headless --screenshot` or `firefox --headless
  --screenshot`), render every slide (`deck.html#N`) at 1280×720 and 1440×900 from a test copy with
  animations off, and **look at each image**. This checks your own output; it is not capturing the
  product.
- Look for: overflow or clipped source lines, callouts off target, elements covering each other,
  captions that don't match the asset, capture text too small to read.
- Spot-check dark theme, a phone width (390×844, which reflows), the JS-off stack and the print
  view. If you changed the shell's script, re-test the keys: → after clicking Next must still
  advance. Fix, then re-render what changed.
- Grep the file for external URLs and for anything the user asked to keep out.
- No browser available: say so in the hand-back, and use numbered points beside an image instead
  of callouts placed over it.

## 7. Deliver

Write the file to `<scratchpad>/presentation-<YYYY-MM-DD>-<slug>.html` unless the user names a
path, and hand it back with the file tools. **Don't publish** unless the user asks — a deck with
someone's words in it stays a local file. Hand back:

- the path and size, and how to navigate (←/→, F, N, print);
- one line per slide;
- **check before you present** — every slide whose asset shows a neighbouring case, every piece of
  example data, every fact you couldn't verify.

## Notes

- **Read-only** is a hard guarantee: this command reads sources and writes one output file, plus
  scratch build files. Rendering its own output is required; capturing the product is not allowed.
- Project-agnostic: the design system, the language and where captures live come from the target
  project's `CLAUDE.md`, never hardcoded here.
- Never invent a quote, a number or a screen. A beat with no source becomes the one question in §1,
  or is marked on its slide as the presenter's own claim.
- Complements `report` (a readable document about things) and `decide` (choices to make). If the
  story is really a pending decision, point at `decide`.
