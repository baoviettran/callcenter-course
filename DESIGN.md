---
name: Call Center Platform: From Click to Conversation
description: A warm-paper technical field manual for a self-hosted call center platform.
colors:
  # Primary — the single accent, applied per course from _base.html
  signal-teal: "#2A7B9B"
  signal-teal-deep: "#1F6280"
  signal-teal-soft: "#E4F2F7"
  signal-teal-muted: "#5A9DB8"

  # Neutral — warm paper ground, ink text, rules
  paper: "#FAF7F2"
  paper-warm: "#F5F0E8"
  surface: "#FFFFFF"
  surface-warm: "#FDF9F3"
  ink: "#2C2A28"
  ink-secondary: "#6B6560"
  ink-muted: "#9E9790"
  rule-line: "#E5DFD6"
  rule-line-light: "#EEEBE5"

  # The instrument slab — code blocks, tooltips, popovers
  instrument-slab: "#1E1E2E"
  instrument-slab-ink: "#CDD6F4"

  # Semantic — quiz verdicts and callouts
  settled-green: "#2D8B55"
  settled-green-soft: "#E8F5EE"
  alert-red: "#C93B3B"
  alert-red-soft: "#FDE8E8"

  # Actors — the named participants in every data-flow diagram
  actor-terracotta: "#D94F30"
  actor-teal: "#2A7B9B"
  actor-violet: "#7B6DAA"
  actor-ochre: "#D4A843"
  actor-moss: "#2D8B55"

  # Syntax — the instrument slab's read-out palette
  syntax-keyword: "#CBA6F7"
  syntax-string: "#A6E3A1"
  syntax-function: "#89B4FA"
  syntax-comment: "#6C7086"
  syntax-number: "#FAB387"
  syntax-property: "#F9E2AF"
  syntax-operator: "#94E2D5"
  syntax-tag: "#F38BA8"
  syntax-text: "#CDD6F4"

typography:
  display:
    fontFamily: "Bricolage Grotesque, Georgia, serif"
    fontSize: "2.25rem"
    fontWeight: 700
    lineHeight: 1.15
  headline:
    fontFamily: "Bricolage Grotesque, Georgia, serif"
    fontSize: "1.5rem"
    fontWeight: 600
    lineHeight: 1.3
  title:
    fontFamily: "Bricolage Grotesque, Georgia, serif"
    fontSize: "1rem"
    fontWeight: 700
    lineHeight: 1.3
  body:
    fontFamily: "DM Sans, -apple-system, sans-serif"
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
  label:
    fontFamily: "JetBrains Mono, Fira Code, Consolas, monospace"
    fontSize: "0.75rem"
    fontWeight: 400
    letterSpacing: "0.1em"
  code:
    fontFamily: "JetBrains Mono, Fira Code, Consolas, monospace"
    fontSize: "0.875rem"
    fontWeight: 400
    lineHeight: 1.7

rounded:
  sm: "8px"
  md: "12px"
  lg: "16px"
  full: "9999px"

spacing:
  "1": "0.25rem"
  "2": "0.5rem"
  "3": "0.75rem"
  "4": "1rem"
  "5": "1.25rem"
  "6": "1.5rem"
  "8": "2rem"
  "10": "2.5rem"
  "12": "3rem"
  "16": "4rem"

components:
  button-primary:
    backgroundColor: "{colors.signal-teal}"
    textColor: "#FFFFFF"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: "8px 20px"
  button-primary-hover:
    backgroundColor: "{colors.signal-teal-deep}"
    textColor: "#FFFFFF"
    rounded: "{rounded.sm}"
    padding: "8px 20px"
  button-secondary:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink-secondary}"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: "8px 20px"
  code-slab:
    backgroundColor: "{colors.instrument-slab}"
    textColor: "{colors.instrument-slab-ink}"
    typography: "{typography.code}"
    padding: "24px"
  pattern-card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink}"
    typography: "{typography.body}"
    rounded: "{rounded.md}"
    padding: "24px"
---

# Design System: Call Center Platform: From Click to Conversation

## Overview

**Creative North Star: "The Annotated Field Manual"**

This is a working engineer's field manual, not a slide deck. The ground is warm off-white paper (`#FAF7F2`) that never reads as a screen; the text is warm charcoal rather than black; and the technical artifacts laid on top of it are hard-edged and cold. That contrast is the whole system. The page is calm and unhurried — generous whitespace, no urgency, nothing demanding attention — while everything *inside* the page is precise and instrumented.

The course teaches a live production call center, so the design's job is to make real system artifacts legible: a real code excerpt in a dark slab, a real data-flow diagram with named actors, a real quiz verdict. Each of these arrives as an object placed on the paper, not as decoration integrated into it. The dark code slab (`#1E1E2E`) is deliberately a different world from the page around it — it is the machine talking, quoted verbatim, with a plain-language translation in warm paper sitting right beside it.

Density is moderate and rhythmic. Every module opens with an oversized ghosted numeral, then proceeds through "screens" (self-contained sections under a Bricolage heading) at a consistent vertical cadence. The learner always knows where they are: a 2px accent progress bar tracks scroll, and seven navigation dots mark module position.

**Confirmed rejection:** the incumbent stylesheet ships a terracotta accent (`#D94F30`) as its template default. That is **not** this product's accent — this course's primary is the teal `#2A7B9B`. See The Per-Course Accent Rule.

**Key Characteristics:**
- Warm paper ground, warm-charcoal ink, no pure white page or pure black text
- One accent (teal) applied through a single per-course override slot
- Two opposed surfaces: calm paper for prose, a dark instrument slab for code
- Warm-tinted shadows at rest (`rgba(44,42,40, …)`), never neutral black
- Monospace-uppercase micro-labels used system-wide as instrumentation

## Colors

A warm neutral page carrying one cool accent and one dark slab. The palette is quiet by design — nothing competes with the code excerpts, which are the only high-contrast surfaces on the page.

### Primary

- **Signal Teal** (`#2A7B9B`): the sole accent, and the system's only saturated color. It marks position (progress bar, active nav dot), identity (module ghost numerals, step numbers), and interaction (selected quiz option, card top borders, callout left borders). **It is applied via `_base.html`, not `styles.css`** — see The Per-Course Accent Rule.
- **Deep Signal Teal** (`#1F6280`): hover state for filled accent elements only. Never used at rest.
- **Soft Signal Teal** (`#E4F2F7`): the tinted fill behind anything accent-adjacent — selected options, active actors, hovered cards, accent callouts.
- **Muted Signal Teal** (`#5A9DB8`): the resting border and underline tone — glossary dashed underlines, visited nav dots, secondary button hover borders. It signals "interactive" without the weight of full accent.

### Neutral

- **Warm Paper** (`#FAF7F2`): the page ground. Carries a barely-visible radial tint at 20%/50% — see Known Drift.
- **Deep Paper** (`#F5F0E8`): recessed fills inside artifacts — actor icon wells, step labels, description plates.
- **Surface White** (`#FFFFFF`): raised cards — quiz containers, pattern cards, flow and architecture panels, chat windows.
- **Warm Surface** (`#FDF9F3`): the reading half of a code-translation block.
- **Ink** (`#2C2A28`): body and heading text.
- **Secondary Ink** (`#6B6560`): supporting prose, captions, step descriptions.
- **Muted Ink** (`#9E9790`): the quietest text — progress counters, zone labels, inactive nav dot borders.
- **Rule Line** (`#E5DFD6`) / **Light Rule** (`#EEEBE5`): borders and dividers, the lighter one for the nav's bottom edge and list rows.

### Semantic

- **Settled Green** (`#2D8B55`) on **Soft Green** (`#E8F5EE`): a correct quiz answer, a correctly placed drag chip.
- **Alert Red** (`#C93B3B`) on **Soft Red** (`#FDE8E8`): an incorrect answer, a misplaced chip, warning callouts.
- **Signal Teal** doubles as the informational callout color, on Soft Signal Teal.

### Instrument slab

**Instrument Slab** (`#1E1E2E`) with **Slab Ink** (`#CDD6F4`): every code block, every tooltip, every nav-dot tooltip. A deep desaturated indigo-charcoal, not black. Its read-out palette (`syntax-*`, Catppuccin-derived) is documented in the frontmatter and rendered in the sidecar.

### Named Rules

**The Per-Course Accent Rule.** The accent is a slot, not a constant. `styles.css` declares a default; `_base.html` overrides `--color-accent`, `--color-accent-hover`, `--color-accent-light`, and `--color-accent-muted` in an inline `:root` block. Any new accent-derived color must be written as a `var()` reference or an alpha of the accent token — **never** as a literal hex or `rgba()`, because literals silently survive a re-theme.

**The Warm Ground Rule.** No pure white page, no pure black text, no neutral-gray shadow. Every neutral is warm (hue ~30–40). A cool neutral in this system reads as a defect.

## Typography

**Display Font:** Bricolage Grotesque (with Georgia, serif)
**Body Font:** DM Sans (with -apple-system, sans-serif)
**Label/Mono Font:** JetBrains Mono (with Fira Code, Consolas, monospace)

**Character:** A contemporary grotesque with real personality set against a neutral, highly legible text face — the display font is allowed to be a character, the body font is not. Bricolage carries every heading, numeral, and actor initial, giving the manual a voice; DM Sans carries all prose and stays out of the way. JetBrains Mono is the instrument: it appears whenever the system itself is speaking — code, file paths, actor names, zone labels, counters.

### Hierarchy

- **Display** (700, 2.25rem / 36px, lh 1.15): module titles. The largest prose-adjacent type on the page. The ghosted module numeral runs above it at 3.75rem / 800 weight / 1.0 lh, in accent at 15% opacity.
- **Headline** (600, 1.5rem / 24px, lh 1.3): screen headings — the section titles inside a module.
- **Title** (700, 1rem / 16px, lh 1.3): component-level titles — pattern card titles, chat window headers.
- **Body** (400, 1rem / 16px, lh 1.6): all prose. Comfortable long-form reading, not compressed.
- **Label** (400, 0.75rem / 12px, 0.1em tracking, uppercase in CSS): the instrument label. Muted ink, monospace, tracked. Appears on diagram zones, file trees, actor names, progress counters, nav tooltips.
- **Code** (400, 0.875rem / 14px, lh 1.7): all code at every size — translation slabs, badge codes, inline snippets.

The scale steps roughly 1.125× per step (`xs` 0.75 → `6xl` 3.75rem), with `4xl`/`5xl`/`6xl` compressed at the 768px and 480px breakpoints so display type never overflows a phone.

### Named Rules

**The Instrument Label Rule.** Any small identifier — a zone name, a file path, an actor label, a step counter, a config key — is monospace, uppercase, letter-spaced 0.1em, and muted ink. This is a system-wide rule, not a diagram-only treatment: it is how the design distinguishes "the system talking" from "the author explaining."

**The One Display Family Rule.** Bricolage Grotesque is for headings, numerals, and initials only. It never sets running prose. If a paragraph is in Bricolage, it is a defect.

## Layout

**Spatial model:** a single centered column, 800px max (`--content-width`), on modules that fill `100dvh` and snap. The nav is a fixed 50px bar spanning the full width with its own 1000px inner column (`--content-width-wide`), so nav content and page content do not share a left edge — the nav is intentionally wider.

**Scroll behavior:** `scroll-snap-type: y proximity` with `scroll-behavior: smooth`. Proximity rather than mandatory, so long modules can be read without fighting the snap. Each module sets `scroll-snap-align: start`.

**Vertical rhythm:** spacing steps from 0.25rem to 4rem on a 4px base. Modules pad `4rem` horizontally and `nav-height + 3rem` from the top so the fixed nav never overlaps content. Screens separate by `4rem` (`--space-16`); blocks inside a screen by `2rem` (`--space-8`) to `1.5rem` (`--space-6`). Sub-heading to first paragraph is consistently `1.5rem`.

**Horizontal rhythm:** cards and panels use `1.5rem` (`--space-6`) to `2rem` (`--space-8`) internal padding. Tight list rows use `0.75rem 1rem` (`--space-3 --space-4`).

**Responsive:** two breakpoints only.
- **768px:** display sizes compress (`4xl` 2.25→1.875rem, `5xl`→2.25rem, `6xl`→3rem). The code-translation block collapses from a 1fr/1fr grid to a single column, and its English half drops the left border for a 3px accent top border.
- **480px:** display sizes compress again (`4xl`→1.5rem). Module padding tightens to `2rem 1rem`. Pattern cards drop to one column. Horizontal flow diagrams rotate: `.flow-steps` becomes a column and `.flow-arrow` rotates 90°.

Multi-column regions (pattern cards, translation blocks, architecture zones) are grid-based with `auto-fit`/`minmax` where the content allows; nothing depends on a fixed column count.

## Elevation & Depth

A **hybrid** system. Surfaces are lifted at rest and lift further on hover — depth is always present, and interaction increases it rather than introducing it. There is no truly flat state for a card.

Every shadow is warm-tinted with the ink color, never neutral black: `rgba(44,42,40, α)`. The warmth is what keeps raised surfaces from reading as a different material from the paper beneath them.

### Shadow Vocabulary

- **Whisper** (`box-shadow: 0 1px 2px rgba(44,42,40,0.05)`): resting depth for the lowest cards — pattern cards, architecture components.
- **Resting** (`box-shadow: 0 4px 12px rgba(44,42,40,0.08)`): the default raised surface — code-translation blocks, quiz containers, flow and architecture panels, chat windows.
- **Raised** (`box-shadow: 0 8px 24px rgba(44,42,40,0.10)`): hover state for cards that lift, and floating tooltips.
- **Float** (`box-shadow: 0 16px 48px rgba(44,42,40,0.12)`): defined in the token set for high-floating surfaces. Currently unreferenced.

The accent also casts a soft glow in two places where an actor is "live" — see Known Drift, because both are still hardcoded to the template's terracotta.

### Named Rules

**The Warm Shadow Rule.** Every shadow is `rgba(44,42,40, α)`. A black or neutral-gray shadow is a defect — it makes a raised surface look like it belongs to a different design.

**The Lift-On-Hover Rule.** Interactive cards translate up 4px and step one shadow level on hover. The lift is the acknowledgement; it is never accompanied by a scale change on cards (only the drag chip scales, because it is being picked up).

## Shapes

**Form language:** soft-rectangular and consistent. Every container is a rounded rectangle; there are no sharp corners anywhere in the system, and no fully-rounded pills except the circular actor/nav markers and drag chips.

- **Small** (`8px`): buttons, badges, tooltips, description plates, list-row items, code chips. The default for anything interactive and compact.
- **Medium** (`12px`): the workhorse — cards, panels, callouts, diagrams, the collapsed participant list. If a container holds content, it is medium.
- **Large** (`16px`): only the biggest single-purpose surfaces — the code-translation block, flow-animation panel, architecture diagram, chat window.
- **Full** (`9999px`): circular only — step numbers (40px), flow actor icons (56px), nav dots (10px), and the pill drag chip.

**Borders are structural, not decorative.** They appear in three roles: a light neutral rule for list rows and the nav underline; a 2px/border-strong for icon wells and actor tiles; and an **accent edge** — a 3px top border on pattern cards and a 4px left border on callouts — which is the system's signature shape motif. That off-center accent edge is how a card declares which kind of thing it is without a label.

**Clipping:** the code-translation block uses `overflow: hidden` to square off its dark slab against the rounded container, so the two-tone split stays crisp at the corners.

## Components

### Buttons

- **Shape:** small radius (8px), 1px neutral border.
- **Primary:** signal teal fill, white text, `8px 20px` padding, DM Sans 600 at 14px. On hover: deep teal fill and a 1px upward translate.
- **Secondary:** white surface, secondary ink text, 1px rule-line border. On hover: border shifts to muted accent and text shifts to accent. It never fills.
- **Text alternative:** quiz reset uses the same secondary treatment; both share one `.btn` base so padding and radius can't drift apart.

### Cards / Containers

- **Corner Style:** medium (12px) for content cards, large (16px) for full-width panels.
- **Background:** surface white on the paper ground; warm surface when the card is a recessed sub-region.
- **Shadow Strategy:** whisper at rest for small cards, resting for full panels (see Elevation & Depth).
- **Border:** pattern cards replace any border with a 3px accent top edge.
- **Internal Padding:** 1.5rem for cards, 2rem for full panels.

### Translation Block

- **The signature component.** A two-column grid: dark instrument slab on the left (real code, syntax-highlighted), warm surface on the right (plain-English reading of each line).
- **Structure:** rows on the English side are separated by 1px rules matching the line count of the code, so the reader can map line to line. The rows carry the accent as a left indicator.
- **Collapse:** below 768px the grid becomes one column and the English half swaps its left border for a 3px accent top border, preserving the "translation attaches to code" reading.

### Quiz

- **Shape:** each option is a medium-radius row with a 1px border and a 20px circular radio at the left.
- **States:** resting (rule-line border) → selected (accent border, soft accent fill, accent radio) → correct (settled green border/fill, green radio with a white inset ring) → incorrect (alert red, same treatment). Disabled options keep their verdict colors and lose the pointer.
- **Radio:** the checked state is a filled circle with a `inset 0 0 0 3px white` ring — the ring is what separates the dot from the fill, and it must follow the state color.
- **Feedback:** a plate below the options, color-matched to the verdict (green / red / teal for the neutral warning). Slides in rather than appearing.

### Navigation

- **Style:** fixed 50px bar, 92% opaque paper with an 8px backdrop blur and a light rule underline. Title in Bricolage 600 at 14px, truncating at 300px.
- **Progress:** a 2px accent bar pinned to the bar's bottom edge, tracking scroll at 100ms linear.
- **Module dots:** 10px circles, transparent with a 2px muted-ink border at rest. Visited = muted accent, filled. Active = full accent, filled, with a 3px soft-accent halo ring. Each reveals a dark-slab tooltip below on hover.
- **Mobile:** the title truncates to 140px; the dots remain the full navigation.

### Flow Animation

- **Actors:** 56px rounded-square wells (medium radius) on warm paper with a 2px border, holding a Bricolage initial, labelled beneath with a mono label.
- **Active state:** the well takes the accent border and soft-accent fill, scales 1.1, and gains a halo ring plus an accent glow.
- **Packet:** a 16px accent dot animated along the wire between actors, with an accent glow.
- **Controls:** playback buttons left, a mono progress counter pushed right.

### Callouts

- **Shape:** medium radius, 32px icon, no shadow. Distinguished by a **4px left border** in one of three colors: accent, info-teal, or alert red, with a matching soft background.
- **Content:** a bolded title line, then secondary-ink body at 14px.

## Do's and Don'ts

### Do:

- **Do** write every accent-derived color as `var(--color-accent*)` or an alpha of the accent token, so a re-theme reaches it.
- **Do** tint every shadow with ink — `rgba(44,42,40, α)` — at one of the four documented steps.
- **Do** set small identifiers (zones, paths, actor names, counters, config keys) in monospace, uppercase, 0.1em tracking, muted ink.
- **Do** give content cards medium (12px) radius and full-width panels large (16px), with 1.5rem or 2rem internal padding respectively.
- **Do** pair every code excerpt with its plain-language translation in the same block — the two-tone split is the course's core teaching device.
- **Do** let Bricolage Grotesque carry headings and numerals only, and DM Sans carry all prose.
- **Do** mark interactive cards with a 4px lift and one shadow step on hover.
- **Do** keep 4xl–6xl display sizes on the documented compression path at 768px and 480px.

### Don't:

- **Don't** introduce a second accent hue. The system has exactly one saturated color; semantic green and red are verdict-only and never used for brand, emphasis, or decoration.
- **Don't** hardcode a hex or `rgba()` where an accent value belongs — that is the exact failure already present in three glow values (see Known Drift).
- **Don't** put a neutral-gray or black shadow on a raised surface.
- **Don't** use pure white as a page background or pure black as text.
- **Don't** set running prose in Bricolage Grotesque.
- **Don't** round a container to `full` — pills are reserved for the circular markers and the drag chip.
- **Don't** use the 16px large radius on a small inline element, or the 8px small radius on a full-width panel.
- **Don't** reduce the accent to a decorative wash. It marks position, identity, and interaction — if it is on screen, it is saying something.

## Known Drift

Three values in `styles.css` predate the per-course accent override and were written as literal terracotta instead of accent references. Under this course's teal they still render orange:

| Location | Value | Should be |
|---|---|---|
| `body` background-image radial tint | `rgba(217,79,48,0.03)` | an alpha of `--color-accent` |
| `.flow-actor.active .flow-actor-icon` glow | `rgba(217,79,48,0.15)` | an alpha of `--color-accent` |
| `.flow-packet` glow | `rgba(217,79,48,0.5)` | an alpha of `--color-accent` |

`--color-actor-1` is also `#D94F30` (the template default accent) and is a genuine actor identity color, not drift — actor 1 is the Browser, and it substitutes cleanly with the accent only if that is intended.
