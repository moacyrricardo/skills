Status: doing
Branch: moacyrricardo/spec-003-rich-html-presentation

# 003 — presentation: a slide-deck command for rich-html

## Context

A hand-prompted run in a private product repo produced an excellent HTML slide deck in one pass
(~7 minutes, no corrections): ten full-screen slides telling a customer-problem → status-quo →
fix → proof story, with verbatim quotes, an annotated screenshot, a "too many steps" path, a
faithful UI miniature, a before/after diagram and an embedded GIF. The user's prompt was eight
lines ("a presentation like `/rich-html:html-doc`, next/previous, full-screen slides, these four
beats, don't re-capture screens"); an orchestrating session expanded it into a brief, and a
second session built the deck after loading `html-doc` (0.3.1).

An audit of both transcripts, the brief, the author's build files and the deck ranked what made
it good:

1. **Curated real sources with exact paths** (transcript, findings, a mockup of the status quo,
   captured stills and a GIF, the component source for exact UI copy). The author read for two
   minutes and searched for none; every slide traces to a source.
2. **The story given as beats with an intended effect** ("it needs to look bureaucratic"), which
   became a countable form (an 8-step path) and the deck's spine (8 → 3).
3. **A beat → form mapping**, partly given, completed by the author.
4. **A self-imposed render-and-look loop** — headless screenshots of every slide, read and fixed —
   that caught a too-small screenshot, misplaced callouts and a caption that overstated its GIF.
   Nobody asked for it.
5. **Honesty devices**: a source line per slide, timestamps on quotes, "example data" labels, a
   note when an asset shows a neighbouring case.

It also shipped defects the loop didn't check: a slide overflowing at 1280×720 under the nav bar,
a popover covering its own chip, Space on a focused button advancing the slide, no print view, no
speaker notes. And it exposed a bug in `html-doc` itself: the canonical theme toggle is a no-op
on its first click when the OS is dark and no `data-theme` is set.

This confirms the earlier finding behind `decide` 0.4.0 (a strong prompt supplies only the goal,
a category → form mapping and real sources) and adds two things for decks: the goal must include
the **arc and its effect**, and the **visual check is a process step**, not luck.

## Decision

Add a fourth rich-html command, **`/rich-html:presentation`**, and fix the `html-doc` toggle.

- **Thin where it can be.** File HOW (self-containment, theme tokens, accessibility, the visual
  evidence forms) comes from `html-doc`. The command owns the **deck shell**, the **slide
  vocabulary**, the **asset pipeline**, the **render check**, and the few places a deck departs
  from `html-doc` (§5 below).
- **Read-only**, like `report`: it reads sources and writes one output file (plus scratch build
  files). It **never re-captures** the product — no app start, no harness, no product
  screenshots. Rendering *its own output* to check it is required and is not capture.
- **Runs in the current session.** The command's framing and gathering steps replace the
  hand-written brief; delegating to another session stays the user's choice.
- **Project-agnostic.** Design system, language, and where captures live come from the target
  project's `CLAUDE.md`.

Resolved questions (user, 2026-11-14):

| # | Question | Decision |
| :-- | :-- | :-- |
| 1 | Speaker notes | Optional per slide; `N` toggles them; shown in print and no-JS |
| 2 | Print / PDF | Yes — all slides stacked, one per page, bars hidden |
| 3 | Real full screen | `F` toggles the Fullscreen API |
| 4 | Fit vs scroll | Fixed 16:9 stage scaled to the window, so a slide always fits; below a narrow width slides stack and scroll |
| 5 | Faithful miniatures | Allowed when no capture exists, built from real markup and copy, one region, captioned as such |
| 6 | Dark theme | The deck surface follows the theme; a design system with no dark palette gets a derived one that passes contrast |
| 7 | Headless browser | Best effort; with none available, say so and use side-by-side numbered points instead of overlay callouts |
| 8 | Delegation | Not built in; runs in the current session |
| 9 | `html-doc` toggle bug | Fixed on this branch, as its own commit |
| 10 | Swipe / click-to-advance | Swipe yes; click-to-advance no (clashes with clicking figures and links) |

## Implementation

### 1. Command file `plugins/rich-html/commands/presentation.md`

Frontmatter `description` + `argument-hint: <the story, as beats> [audience / occasion /
sources]`. Same voice and section rhythm as `report` / `decide`. Target ≤ 220 lines.

### 2. Frame — beats, effect, audience

- **Beats**: the user's ordered points. Keep the order; the command chooses how many slides each
  beat gets.
- **Effect**: what each beat should make the audience conclude. Turn an adjective into a
  countable form ("bureaucratic" → numbered steps with a tally).
- **Audience and occasion**: presented live vs sent to be read (sets words per slide and whether
  notes matter).
- State the reading and proceed; one question only if a beat has no source.

### 3. Gather — real sources, looked at

- Sources: transcripts/messages, specs and docs, PRs and their attachments, existing captures
  (screenshots, GIFs), code for exact UI copy. `CLAUDE.md` says where captures live.
- Read every text you will quote; **open every image and sample GIF frames before writing about
  them**.
- Quotes verbatim with a locator (timestamp, line, URL); note transcription doubt; never translate
  a quote silently. No source → no quote.
- Example data in a figure is labelled as example; a counted claim lists what it counts.

### 4. Slides — one claim, one form

- Headline = the slide's claim as a sentence (leaky/pure example pair: a topic label vs a claim,
  generic). Overline = which beat.
- One figure per slide, always visible; body ≤ ~120 words; a source line at the foot.
- Beat → form table (illustrations, not a closed list): what someone said → hero quote + up to two
  supporting; how a screen looks today → existing capture with ≤ 3 numbered callouts; a path that
  is too long → numbered step cards with where each happens, plus a tally; a control with no
  capture → UI miniature from real markup/copy; a change → before/after on one baseline, delta
  highlighted; it working → captured GIF/still with numbered captions; a trend → chart (load
  `dataviz`); the takeaway → one number or sentence.
- One cover (title, one-line lead, beats as agenda) and one closing summary. No dividers,
  "questions?" or thank-you slides; no decorative figures.

### 5. Deck shell and the departures from `html-doc`

- Structure: `<header>` (title, theme button), `<main>` of `<section class="slide">`, `<nav>`
  (Previous, `N / total`, Next). `h1` on the cover, `h2` per slide.
- Navigation: buttons; ← / →, PageUp / PageDown, Space / Shift+Space, Home / End; swipe; `F`
  full screen; `N` notes. Key handlers ignore events whose target is a button, input or link, and
  any modifier combo. Counter `aria-live`. Progress bar. `#N` in the URL, read on load and on
  `hashchange`.
- Fit: a fixed 16:9 stage scaled to the window; below a narrow breakpoint slides stack.
- JS never gates content: slides are hidden only under a `.js` class the script sets; without JS
  they stack in order with notes visible. Print uses the same stacked view, one slide per page.
- Departures, stated in the command:
  - no `<details>` on slides — the sequence is the disclosure; extra proof goes to notes;
  - figures are promoted to the visible body (the `html-doc` §4 promotion clause), one per slide;
  - captured or mimicked product UI keeps its native colours inside a framed figure; only the
    deck surface follows the theme;
  - a miniature may be faithful when built from real markup and copy (Decision #5).
- Design system from `CLAUDE.md` when present (tokens, type, reserved colours); otherwise
  `html-doc`'s neutral default.

### 6. Assets

- Crop to the region the slide is about; downscale to displayed width; JPEG for captures; cap
  GIF size and frames. Budget ~5 MB; report the final size.
- Keep base64 out of the model's context: write the deck with placeholders and inline with a small
  script that fails if any placeholder is left.
- `alt` on every image; callout numbers match the points beside the image.

### 7. Check — render and look

- Render every slide headless at 1280×720 and 1440×900 from a test copy with animation off; read
  each image. Check: overflow, clipped footers, callouts off target, overlaps, captions that don't
  match the asset, unreadable capture text.
- Spot-check dark theme, a narrow width, the JS-off stack and print. Fix and re-render.
- Grep the file for external URLs and anything the user said to keep out.

### 8. Deliver

- Scratchpad unless the user names a path; never publish unless asked (a deck with customer words
  stays local).
- Hand back: path, size, how to navigate, one line per slide, and a "check before you present"
  list (assets showing a neighbouring case, example data, unverified facts).

### 9. `html-doc` toggle fix (separate commit)

The canonical toggle computes the effective theme (`data-theme` or `prefers-color-scheme`) before
flipping, and the dark media block is scoped `:root:not([data-theme="light"])`, so the first
click always switches.

### 10. Plugin + docs

- `plugin.json`: 0.4.0 → **0.5.0**; description names four commands.
- `html-doc` intro and `docs/rich-html-guide.{md,html}`: list `presentation` alongside the others,
  a section for it, "when to use which", changelog entry.
- `CATALOG.md` row for 003.

## Verification

A blind A/B, as for `decide` 0.4.0: a synthetic project with a story and real-looking sources
(fictional — no private data), built twice by the same model — once from a one-line
`/rich-html:presentation <beats>` with the new command, once with only `html-doc` and the
original eight-line prompt — then judged against a rubric: one claim per slide, beat → form fit,
verbatim sourced quotes, no re-capture, fit at 1280×720, navigation and print, no-JS stack, size,
and the hand-back list.

## Known gaps

- The command can't make a weak story good; it can only refuse to invent sources for it.
- Visual check depends on a local headless browser; without one, quality relies on markup alone.
- Faithful miniatures sit close to `prototype:mockup`; the line is "one region, captioned, inside a
  deck" vs "a clickable facsimile".
