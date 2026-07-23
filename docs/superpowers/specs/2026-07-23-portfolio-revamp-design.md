# Portfolio Revamp — Design Spec

**Date:** 2026-07-23
**Repo:** `yann-b-b.github.io` (static GitHub Pages site)
**Status:** Approved, ready for implementation planning

## Goal

Revamp the personal portfolio to (1) look more polished and modern and (2) grow
from a project wall into a fuller personal site. Primary audience: ML/research
recruiters and hiring managers (the same audience the tailored resumes target).

## Decisions (locked during brainstorming)

- **Aesthetic:** Editorial — warm-paper background, serif display headings, sans
  body, single deep-rust accent.
- **Theme:** Light only. No dark mode, no toggle.
- **Layout:** Single page with a slim sticky top nav that smooth-scrolls to
  sections. No build step, no framework, no Jekyll.
- **New sections:** expanded About, and a distinct Research/Publications section.
- **No blog / no CMS** (explicitly deferred; add a lightweight "Notes" shelf later
  only if real posts get written).

## Visual System

- **Palette:** paper `#f7f4ef`, ink `#1c1917`, rust accent `#7c2d12`, muted text
  `#78716c`, hairline borders `#e7e0d5`.
- **Type:** display serif for headings (Fraunces or Newsreader via Google Fonts);
  Inter or system-sans for body. Graceful fallback to Georgia / system stack if
  the web fonts fail to load.
- **Retire:** floating-cogs background, blue overlay, rainbow-shadow hover on the
  intro, dark `#333` header. Replace card hover with a subtle lift + rust
  underline (no rainbow animation).

## Page Structure (top to bottom)

1. **Sticky nav** — name on the left; `About · Research · Projects` anchor links
   and a `View CV` button on the right. Smooth-scroll to sections. Collapses
   gracefully on mobile.
2. **Hero strip** — name, one-line tagline ("ML & AI researcher — multimodal, NLP,
   on-device"), and email / LinkedIn / GitHub icons.
3. **About** — profile photo alongside:
   - *Current focus* paragraph (multimodal ML, NLP/LLM systems, research toward a
     manuscript-in-prep).
   - *Education* line (degree(s) + school(s)).
   - *Interests / personal* line (a touch of personality).
   - **Animated skills marquee** — a horizontal row of tech tags (Python, PyTorch,
     HuggingFace, RAG, Gaussian Processes, TensorFlow Lite, …) that scrolls
     sideways and loops infinitely, wrapping seamlessly, pausing on hover.
4. **Research (Selected)** — a distinct, heavier-weight section featuring the GP
   Fusion work (manuscript in preparation) and the Hybrid RAG work, separated
   visually from course projects because this is what research recruiters scan for.
5. **Projects** — the existing project cards, restyled to the editorial look, kept
   grouped by term (Spring 2026 → Spring 2023). All current projects retained,
   copy/metrics/images reused verbatim.
6. **Footer** — name, year (corrected to 2026), quick links.

## Content Sourcing

- **Project cards:** reuse existing copy, metrics, images, and links exactly as
  they appear in the current `index.html`. Nothing invented.
- **About copy** (current focus, education, interests): drafted from
  `accomplishments.md` and the resume repo's `resume.tex`, then reviewed and
  approved by Yann before shipping. No fabricated facts or metrics.
- **CV link:** unchanged — points at `/Yann_Baglin-Bunod_CV.pdf` (already updated
  to the Samsung-tailored resume).

## Technical Approach

- Stay a **hand-written single `index.html`**, but split the growing styles into a
  separate **`styles.css`** for maintainability. Keep it trivially hostable on
  GitHub Pages.
- **Responsive:** nav condenses on small screens; About photo and content stack;
  project grid reflows to one column.
- **Accessibility/motion:** the skills marquee respects
  `prefers-reduced-motion: reduce` (animation disabled, tags shown static and
  wrapped). Sufficient color contrast on paper background. Nav links are real
  anchors.
- **Fonts:** loaded via Google Fonts `<link>`; site remains fully functional with
  system-serif/sans fallback if the request is blocked.

## Out of Scope

- Blog / Markdown posts / Jekyll.
- Dark mode or theme toggle.
- Any CMS or dynamic backend.
- Rewriting project descriptions or adding new projects (restyle only).

## Success Criteria

- Editorial light look implemented; old cog/rainbow styling gone.
- Sticky nav smooth-scrolls to About, Research, Projects.
- About shows focus + education + interests + a working infinite skills marquee
  (that stills under reduced-motion).
- Research section visually distinct from the project grid.
- All existing projects present and correctly linked.
- Responsive down to mobile; CV link works; footer year says 2026.
- No invented content; About copy approved before release.
