# Experience Timeline — Design Spec

**Date:** 2026-07-23
**Repo:** `yann-b-b.github.io`
**Status:** Approved, ready for implementation planning

## Goal

Add an auto-scrolling "Experience" timeline section to the portfolio — a vertical
motif echoing the skills banner, showing work history oldest → newest, that drifts
upward, pauses on hover/focus, and stays readable and accessible.

## Decisions (locked during brainstorming)

- **Motion:** auto-scroll upward, continuous, **pauses on hover/focus**; under
  `prefers-reduced-motion: reduce` it stops and shows all entries statically.
- **Layout:** **left-rail timeline** — a vertical line with a rust node dot per
  entry; small boxes stacked to its right.
- **Box content:** minimal — **year · role · org** only (no per-role descriptions,
  no tech tags).
- **Placement:** new `#experience` section directly after `#about` (order: nav →
  hero → About → Experience → Research → Projects → footer). Nav gains an
  "Experience" link.
- **Order:** oldest at top → newest at bottom.

## Entries (verbatim from `resume.tex`, no invented content)

| Year | Role | Org |
|------|------|-----|
| 2023 | Instructional Assistant, ECE 15 (C Programming) | UC San Diego |
| 2024 | Summer Research Intern (Medical Swallow Analysis) | UC San Diego |
| 2025 | Data Science Intern | OpenAirlines, Toulouse |
| 2026 | ML Researcher (Gaussian Process Fusion) | Schmidt AI in Science, UCSD |

## The Component

- A **fixed-height window** (~3 entries tall) with `overflow: hidden` and a
  **vertical fade mask** (top + bottom transparent) — the vertical analogue of the
  skills banner's horizontal edge fade.
- Inside, a **track** that holds the entry list **duplicated** and animates
  `translateY` to loop. The banner's hard-won lesson applies verbatim: **one copy
  of the list must be taller than the window**, or a gap/stutter appears. Guarantee
  it by (a) capping the window height below one copy's height and (b) using enough
  duplicate copies that one animation half far exceeds the window.
- **Pause on hover/focus**: `animation-play-state: paused` on `:hover`/`:focus-within`.
- **Reduced motion**: `@media (prefers-reduced-motion: reduce)` sets
  `animation: none`, shows only the first (non-duplicated) copy, removes the mask,
  and lets the section size to content so all four entries are visible and static.
- **Left rail:** a vertical line (hairline) running through rust node dots aligned
  to each entry; boxes to the right show the year (serif), role, and org.

## Styling

- Reuse the editorial tokens: paper `#f7f4ef`, ink `#1c1917`, rust `#7c2d12`
  (node dots + year accent), muted `#78716c`, hairline `#e7e0d5`. Year label in
  the display serif; role/org in the sans body font.
- Section constrained to the content width via the existing `.wrap`.

## Accessibility & Integrity

- The section is real content (not `aria-hidden`). The **duplicated loop copy is
  `aria-hidden="true"`** so assistive tech reads each role once.
- No fabricated content: every entry is drawn from `resume.tex`. Titles match the
  resume; the parenthetical project names ("Medical Swallow Analysis", "Gaussian
  Process Fusion") are kept for clarity and are also from the resume.
- Respects `prefers-reduced-motion`; page body never scrolls horizontally; the
  section is responsive (window height/box padding scale down on mobile).

## Technical Approach

- Static site, no build step. Modify `index.html` (nav link + new `#experience`
  section) and append CSS to `styles.css`.
- Vertical marquee via CSS `@keyframes` on `translateY`; no JavaScript.

## Out of Scope

- Per-role descriptions, tech tags, logos, or location beyond the org line.
- Clickable/expandable entries, filtering, or links out.
- Any change to existing sections beyond adding the nav link.

## Success Criteria

- `#experience` appears after About; nav "Experience" link scrolls to it.
- Four entries render oldest → newest in a left-rail timeline with node dots.
- The list auto-scrolls upward and **loops with no visible gap or stutter at any
  screen width** (one copy taller than the window; enough copies).
- Hover/focus pauses it; reduced-motion shows all four static with no animation.
- Editorial styling consistent; all content from `resume.tex`; no horizontal
  overflow; responsive on mobile.
