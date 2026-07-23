# Portfolio Revamp Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the single-page portfolio in an editorial, light-theme look with a sticky-nav layout, an expanded About (with an animated skills marquee), and a distinct Research section — reusing all existing project content.

**Architecture:** Stays a hand-written static site on GitHub Pages: one `index.html` for structure and a new `styles.css` for all styling (extracted from the current inline `<style>`). No framework, no build step, no Jekyll. Content (project cards, images, PDFs) is reused verbatim from the current site; the GP Fusion and Hybrid RAG cards are promoted into a featured Research section.

**Tech Stack:** HTML5, CSS3 (custom properties, flexbox/grid, CSS keyframe animation), Google Fonts with system fallback. No JS framework; a few lines of vanilla JS only if smooth-scroll needs a polyfill (native `scroll-behavior: smooth` is preferred).

## Global Constraints

- Static site only — no build step, no Jekyll, no framework, no backend.
- Light theme only — no dark mode, no theme toggle.
- Palette: paper `#f7f4ef`, ink `#1c1917`, rust accent `#7c2d12`, muted `#78716c`, hairline `#e7e0d5`.
- Display serif for headings (Fraunces), Inter for body, both via Google Fonts; must fall back to Georgia / system-sans if fonts fail to load.
- No invented content: project cards reuse existing copy/metrics/images/links verbatim; About copy uses only real facts from `resume.tex` / `accomplishments.md`; the interests line requires Yann-provided text.
- All existing media (`media/`) and PDFs (`assets/`) must remain referenced and working.
- Skills marquee must respect `prefers-reduced-motion: reduce` (static, wrapped, no animation).
- Responsive down to mobile: nav condenses, About stacks, project grid reflows to one column; page body never scrolls horizontally.
- CV link stays `/Yann_Baglin-Bunod_CV.pdf`. Footer year is 2026.

## File Structure

- **Modify:** `index.html` — replace the inline `<style>` block with a `<link>` to `styles.css` + Google Fonts; restructure `<body>` into semantic sections: `nav`, hero, `#about`, `#research`, `#projects`, `footer`.
- **Create:** `styles.css` — all styling: design tokens (`:root` custom properties), base/reset, typography, nav, hero, about + marquee, research, project grid, footer, responsive media queries, reduced-motion block.
- **Unchanged:** everything in `media/` and `assets/`; `Yann_Baglin-Bunod_CV.pdf`.

Verification for this static site is manual/observable (open the page, confirm outcomes) plus an HTML well-formedness check. There is no unit-test framework; each task ends with a concrete check list and a commit.

**Reference — real facts to use (from `resume.tex`):**
- Education: `M.S. in Electrical & Computer Engineering — Machine Learning & Data Science, UC San Diego (2026)`; `B.S. in Electrical Engineering, UC San Diego, GPA 3.75`.
- Skills tags: Python, C++17, Kotlin, C, MATLAB, PyTorch, TensorFlow, TensorFlow Lite, Scikit-learn, Hugging Face, NumPy, Pandas, FAISS.
- Research affiliation (already on site): "Research with K. Polyzos, Schmidt AI in Science (UCSD); manuscript in preparation."

---

### Task 1: CSS foundation + document shell

**Files:**
- Create: `styles.css`
- Modify: `index.html` (replace `<head>` `<style>` with links; strip cog background + overlay)

**Interfaces:**
- Produces: the `:root` design tokens (`--paper`, `--ink`, `--rust`, `--muted`, `--hairline`, `--font-serif`, `--font-sans`) and base typography that every later task's markup relies on. Body background is `var(--paper)`; headings use `var(--font-serif)`.

- [ ] **Step 1: Create `styles.css` with tokens, reset, and base typography**

```css
:root {
  --paper: #f7f4ef;
  --ink: #1c1917;
  --rust: #7c2d12;
  --muted: #78716c;
  --hairline: #e7e0d5;
  --font-serif: "Fraunces", Georgia, "Times New Roman", serif;
  --font-sans: "Inter", -apple-system, "Segoe UI", Roboto, sans-serif;
  --maxw: 1100px;
}

* { box-sizing: border-box; }

html { scroll-behavior: smooth; }

body {
  margin: 0;
  background: var(--paper);
  color: var(--ink);
  font-family: var(--font-sans);
  font-size: 17px;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3 { font-family: var(--font-serif); font-weight: 600; line-height: 1.15; letter-spacing: -0.01em; }

a { color: var(--rust); text-decoration: none; }
a:hover { text-decoration: underline; }

img { max-width: 100%; }

.wrap { max-width: var(--maxw); margin: 0 auto; padding: 0 1.5rem; }

.eyebrow {
  font-family: var(--font-sans);
  font-size: 0.72rem;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--muted);
}
```

- [ ] **Step 2: Replace the `<head>` of `index.html`**

Remove the entire inline `<style>...</style>` block and the cog `background-image` / blue `html::before` overlay. Replace the head so it links the fonts and stylesheet:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Yann Baglin-Bunod — ML &amp; AI Researcher</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500;9..144,600&family=Inter:wght@400;500;600&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="styles.css">
</head>
```

- [ ] **Step 3: Verify the shell renders**

Run: `open index.html` (or reload the GitHub Pages preview).
Expected: warm paper background (no floating cogs, no blue tint); any existing text now renders in Inter with serif headings; no console errors for the stylesheet. If offline and fonts are blocked, headings fall back to Georgia and body to system-sans (still styled).

- [ ] **Step 4: Validate HTML well-formedness**

Run: `python3 -c "import html.parser,sys; \nclass P(html.parser.HTMLParser):\n  pass\nP().feed(open('index.html').read()); print('parsed-ok')"`
Expected: prints `parsed-ok` with no exception.

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css
git commit -m "Extract styles to styles.css; editorial base + remove cog theme"
```

---

### Task 2: Sticky nav + hero

**Files:**
- Modify: `index.html` (replace the old `<header>` with a sticky `<nav>` and a hero `<header>`)
- Modify: `styles.css` (append nav + hero styles)

**Interfaces:**
- Consumes: tokens + `.wrap` from Task 1.
- Produces: anchor targets `#about`, `#research`, `#projects` used by later tasks; the `.nav` sticky bar and `.hero` block.

- [ ] **Step 1: Replace the `<header>` block in `index.html`**

```html
<nav class="nav">
  <div class="wrap nav-inner">
    <a class="nav-name" href="#top">Yann Baglin-Bunod</a>
    <div class="nav-links">
      <a href="#about">About</a>
      <a href="#research">Research</a>
      <a href="#projects">Projects</a>
      <a class="nav-cv" href="/Yann_Baglin-Bunod_CV.pdf" target="_blank" rel="noopener">View CV</a>
    </div>
  </div>
</nav>

<header id="top" class="hero">
  <div class="wrap">
    <p class="eyebrow">Machine Learning &amp; AI</p>
    <h1 class="hero-title">Yann Baglin-Bunod</h1>
    <p class="hero-tagline">ML &amp; AI researcher — multimodal, NLP, and on-device systems.</p>
    <div class="hero-social">
      <a href="mailto:ybaglinbunod@gmail.com">Email</a>
      <a href="https://www.linkedin.com/in/yann-baglin-bunod" target="_blank" rel="noopener">LinkedIn</a>
      <a href="https://github.com/Yann-b-b" target="_blank" rel="noopener">GitHub</a>
    </div>
  </div>
</header>
```

- [ ] **Step 2: Append nav + hero styles to `styles.css`**

```css
.nav {
  position: sticky; top: 0; z-index: 50;
  background: rgba(247,244,239,0.85);
  backdrop-filter: saturate(1.2) blur(8px);
  border-bottom: 1px solid var(--hairline);
}
.nav-inner { display: flex; align-items: center; justify-content: space-between; height: 58px; }
.nav-name { font-family: var(--font-serif); font-weight: 600; color: var(--ink); font-size: 1.05rem; }
.nav-links { display: flex; align-items: center; gap: 1.4rem; }
.nav-links a { color: var(--ink); font-size: 0.92rem; }
.nav-links a:hover { color: var(--rust); text-decoration: none; }
.nav-cv {
  background: var(--rust); color: #fff !important;
  padding: 0.4rem 0.85rem; border-radius: 6px; font-weight: 500;
}
.nav-cv:hover { background: #symbol; filter: brightness(1.1); }

.hero { padding: 4.5rem 0 3rem; border-bottom: 1px solid var(--hairline); }
.hero-title { font-size: clamp(2.4rem, 6vw, 4rem); margin: 0.3rem 0 0.6rem; }
.hero-tagline { font-size: 1.2rem; color: var(--muted); margin: 0 0 1.4rem; max-width: 40ch; }
.hero-social { display: flex; gap: 1.2rem; font-size: 0.95rem; }
```

Note: replace the invalid `#symbol` placeholder — set `.nav-cv:hover { filter: brightness(1.15); }` and drop the `background:` line on hover.

- [ ] **Step 3: Verify nav + hero**

Run: `open index.html`
Expected: a slim sticky bar stays pinned at the top when scrolling; clicking About/Research/Projects smooth-scrolls (targets exist after later tasks — for now they may not jump yet); the hero shows name, tagline, and three working social links; "View CV" opens the PDF.

- [ ] **Step 4: Commit**

```bash
git add index.html styles.css
git commit -m "Add sticky nav and hero"
```

---

### Task 3: About section with animated skills marquee

**Files:**
- Modify: `index.html` (add `#about` section after the hero)
- Modify: `styles.css` (append about + marquee styles + reduced-motion block)

**Interfaces:**
- Consumes: tokens, `.wrap`, `.eyebrow`.
- Produces: `#about` anchor target; `.marquee` component.

- [ ] **Step 1: Confirm About copy with Yann before writing it**

The *current focus* and *education* lines below use real facts from `resume.tex`. The **interests line requires Yann's own sentence** — ask him for one line (e.g. what he does outside research). Do not invent it; omit the interests `<p>` until he provides text.

Draft to confirm:
- Focus: "I'm an M.S. student and ML/AI researcher at UC San Diego. My work spans multimodal representation learning, NLP and LLM systems, and on-device inference — currently a Gaussian-process fusion study (manuscript in preparation) and a hybrid retrieval pipeline over technical documents."
- Education: "M.S. in Electrical & Computer Engineering (Machine Learning & Data Science), UC San Diego, 2026 · B.S. in Electrical Engineering, UC San Diego (GPA 3.75)."

- [ ] **Step 2: Add the `#about` section to `index.html`**

Insert after the `</header>` hero. The marquee track lists the tags **twice** back-to-back — this is what makes the loop seamless (the animation translates by exactly -50%).

```html
<section id="about" class="about">
  <div class="wrap about-grid">
    <img class="about-photo" src="media/profile_pic.jpg" alt="Yann Baglin-Bunod">
    <div class="about-text">
      <p class="eyebrow">About</p>
      <p>I'm an M.S. student and ML/AI researcher at UC San Diego. My work spans multimodal representation learning, NLP and LLM systems, and on-device inference — currently a Gaussian-process fusion study (manuscript in preparation) and a hybrid retrieval pipeline over technical documents.</p>
      <p class="about-edu"><strong>Education:</strong> M.S. in Electrical &amp; Computer Engineering (Machine Learning &amp; Data Science), UC San Diego, 2026 · B.S. in Electrical Engineering, UC San Diego (GPA 3.75).</p>
      <!-- interests <p> added only once Yann provides the sentence -->
    </div>
  </div>
  <div class="marquee" aria-label="Skills and tools">
    <div class="marquee-track">
      <span>Python</span><span>C++17</span><span>Kotlin</span><span>C</span><span>MATLAB</span>
      <span>PyTorch</span><span>TensorFlow</span><span>TensorFlow Lite</span><span>Scikit-learn</span>
      <span>Hugging Face</span><span>NumPy</span><span>Pandas</span><span>FAISS</span>
      <span>Python</span><span>C++17</span><span>Kotlin</span><span>C</span><span>MATLAB</span>
      <span>PyTorch</span><span>TensorFlow</span><span>TensorFlow Lite</span><span>Scikit-learn</span>
      <span>Hugging Face</span><span>NumPy</span><span>Pandas</span><span>FAISS</span>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Append about + marquee styles to `styles.css`**

```css
.about { padding: 3.5rem 0 2rem; }
.about-grid { display: grid; grid-template-columns: 160px 1fr; gap: 2rem; align-items: start; }
.about-photo { width: 160px; height: 160px; border-radius: 10px; object-fit: cover; }
.about-text p { margin: 0.4rem 0 0.9rem; }
.about-edu { color: var(--muted); font-size: 0.95rem; }

.marquee {
  margin-top: 2rem; overflow: hidden;
  border-top: 1px solid var(--hairline); border-bottom: 1px solid var(--hairline);
  padding: 0.9rem 0;
  -webkit-mask-image: linear-gradient(90deg, transparent, #000 8%, #000 92%, transparent);
          mask-image: linear-gradient(90deg, transparent, #000 8%, #000 92%, transparent);
}
.marquee-track { display: inline-flex; gap: 0.9rem; white-space: nowrap; animation: marquee 28s linear infinite; }
.marquee:hover .marquee-track { animation-play-state: paused; }
.marquee-track span {
  font-family: var(--font-sans); font-size: 0.9rem; color: var(--ink);
  background: #fff; border: 1px solid var(--hairline); border-radius: 999px; padding: 0.35rem 0.9rem;
}
@keyframes marquee { from { transform: translateX(0); } to { transform: translateX(-50%); } }

@media (prefers-reduced-motion: reduce) {
  .marquee { -webkit-mask-image: none; mask-image: none; }
  .marquee-track { animation: none; flex-wrap: wrap; white-space: normal; }
}
```

- [ ] **Step 4: Verify the About section and marquee**

Run: `open index.html`
Expected: photo + focus + education render in two columns; the skills row scrolls continuously right-to-left and loops with **no visible jump or gap** at the seam; hovering the row pauses it. Then test reduced motion — macOS: System Settings → Accessibility → Display → Reduce motion (or DevTools "Emulate prefers-reduced-motion"): the row stops animating and wraps to multiple static lines.

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css
git commit -m "Add About section with animated skills marquee"
```

---

### Task 4: Research (Selected) section

**Files:**
- Modify: `index.html` (add `#research` section; move GP Fusion + Hybrid RAG cards out of the old grid)
- Modify: `styles.css` (append research styles)

**Interfaces:**
- Consumes: tokens, `.wrap`, `.eyebrow`.
- Produces: `#research` anchor; `.research-item` layout reused by both featured entries.

- [ ] **Step 1: Add the `#research` section to `index.html`**

Reuse the exact copy/metrics/images currently on the GP Fusion and RAG cards (from the current `index.html`). Render them heavier than project cards — full-width rows with the image beside the text.

```html
<section id="research" class="research">
  <div class="wrap">
    <p class="eyebrow">Selected Research</p>
    <h2 class="section-title">Research</h2>

    <article class="research-item">
      <img src="media/gp_fusion.png" alt="Multimodal Gaussian Process Fusion">
      <div>
        <h3>Multimodal Gaussian Process Fusion</h3>
        <p>Ran a 9,703-configuration ablation of multimodal GP fusion (9 embedders × 3 GP scaling methods × 5 ensemble modes); a data-pipeline fix raised best test R² from 0.629 to 0.662. Built a spectrally-normalized MLP over Gemini text embeddings to extract a 128-d GP input, reaching test R² 0.6685 / NLPD 0.57.</p>
        <p class="research-meta">Research with K. Polyzos, Schmidt AI in Science (UCSD) · manuscript in preparation</p>
      </div>
    </article>

    <article class="research-item">
      <img src="media/rag_context.png" alt="Hybrid RAG results table">
      <div>
        <h3>Hybrid RAG over Technical Docs</h3>
        <p>Engineered a hybrid RAG pipeline — hierarchical chunking, bge-m3 dense + BM25 sparse fused by reciprocal-rank fusion, then a cross-encoder reranker — reaching F1@5 0.853 / Precision@5 0.889. Designed a HyDE query stage with parent-chunk expansion and a 1,700-token context packer.</p>
        <p class="research-meta">Team project (2 people) · <a href="assets/RAG_Context_Engineering_Report.pdf" target="_blank" rel="noopener">Read the paper</a></p>
      </div>
    </article>
  </div>
</section>
```

- [ ] **Step 2: Append research styles to `styles.css`**

```css
.research { padding: 3rem 0; border-top: 1px solid var(--hairline); }
.section-title { font-size: 2rem; margin: 0.2rem 0 1.6rem; }
.research-item { display: grid; grid-template-columns: 300px 1fr; gap: 1.8rem; align-items: start; padding: 1.6rem 0; border-top: 2px solid var(--ink); }
.research-item:first-of-type { border-top: 2px solid var(--ink); }
.research-item img { border-radius: 8px; border: 1px solid var(--hairline); }
.research-item h3 { font-size: 1.4rem; margin: 0 0 0.5rem; }
.research-meta { color: var(--muted); font-size: 0.9rem; margin-top: 0.7rem; }
```

- [ ] **Step 3: Verify the Research section**

Run: `open index.html`
Expected: a "Research" section shows two featured rows (GP Fusion, Hybrid RAG) with images beside text, visually heavier than project cards; the RAG paper link opens the PDF; these two no longer appear in the project grid (removed there in Task 5). Nav "Research" link scrolls here.

- [ ] **Step 4: Commit**

```bash
git add index.html styles.css
git commit -m "Add featured Research section (GP Fusion, Hybrid RAG)"
```

---

### Task 5: Projects grid restyle

**Files:**
- Modify: `index.html` (wrap remaining cards in `#projects`; delete the two promoted cards; restyle markup)
- Modify: `styles.css` (append project grid + card styles)

**Interfaces:**
- Consumes: tokens, `.wrap`, `.eyebrow`.
- Produces: `#projects` anchor; `.project-card` styling.

- [ ] **Step 1: Restructure the projects markup in `index.html`**

Keep every remaining project verbatim (Flashcard Maker, Multimodal Embedding Alignment, NLP LLM→SLM, Chess Bot RL, Smart Pillow, Tweet Classification, Swallow Detection, ASL Recognition, Gripper Arm) with their exact images, copy, term labels, and links. Wrap them:

```html
<section id="projects" class="projects-section">
  <div class="wrap">
    <p class="eyebrow">Coursework &amp; Side Projects</p>
    <h2 class="section-title">Projects</h2>
    <div class="projects-grid">
      <!-- existing .project-card blocks, minus GP Fusion and RAG (now in Research) -->
    </div>
  </div>
</section>
```

Remove the old `.projects` wrapper's inline term comments only if they clutter; keep the `(Spring 2025)` etc. term spans inside each card's `<h2>`.

- [ ] **Step 2: Append project grid + card styles to `styles.css`**

```css
.projects-section { padding: 3rem 0; border-top: 1px solid var(--hairline); }
.projects-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 1.6rem; }
.project-card {
  background: #fff; border: 1px solid var(--hairline); border-radius: 10px; padding: 1.1rem;
  transition: transform 0.18s ease, box-shadow 0.18s ease;
}
.project-card:hover { transform: translateY(-4px); box-shadow: 0 10px 24px rgba(28,25,23,0.10); }
.project-card img, .project-card video { width: 100%; border-radius: 6px; margin-bottom: 0.8rem; }
.project-card h2 { font-size: 1.15rem; margin: 0 0 0.5rem; }
.project-card ul { margin: 0 0 0.6rem; padding-left: 1.1rem; font-size: 0.92rem; }
.project-card a { font-weight: 500; }
.project-card a:hover { text-decoration: underline; text-decoration-color: var(--rust); }
```

- [ ] **Step 3: Verify projects grid**

Run: `open index.html`
Expected: all nine remaining projects render as editorial cards in a responsive grid; GP Fusion and RAG are absent here (they live in Research); every image loads and every link/PDF (`assets/*.pdf`, GitHub, Kaggle, Canva, Google Slides) still resolves; hover gives a subtle lift (no rainbow). Cross-check the card count against the current site: 11 original − 2 promoted = 9 cards.

- [ ] **Step 4: Commit**

```bash
git add index.html styles.css
git commit -m "Restyle projects into editorial grid"
```

---

### Task 6: Footer, responsive polish, and final verification

**Files:**
- Modify: `index.html` (footer)
- Modify: `styles.css` (footer + responsive media queries)

**Interfaces:**
- Consumes: everything above.

- [ ] **Step 1: Replace the footer in `index.html`**

```html
<footer class="footer">
  <div class="wrap footer-inner">
    <span>© 2026 Yann Baglin-Bunod</span>
    <span class="footer-links">
      <a href="mailto:ybaglinbunod@gmail.com">Email</a>
      <a href="https://www.linkedin.com/in/yann-baglin-bunod" target="_blank" rel="noopener">LinkedIn</a>
      <a href="https://github.com/Yann-b-b" target="_blank" rel="noopener">GitHub</a>
    </span>
  </div>
</footer>
```

- [ ] **Step 2: Append footer + responsive styles to `styles.css`**

```css
.footer { border-top: 1px solid var(--hairline); padding: 2rem 0; margin-top: 2rem; color: var(--muted); font-size: 0.9rem; }
.footer-inner { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem; }
.footer-links a { margin-left: 1.1rem; }

@media (max-width: 720px) {
  .nav-links { gap: 0.9rem; font-size: 0.85rem; }
  .nav-links a:not(.nav-cv) { display: none; }         /* keep only CV button on small screens */
  .about-grid { grid-template-columns: 1fr; }
  .about-photo { width: 120px; height: 120px; }
  .research-item { grid-template-columns: 1fr; }
  .projects-grid { grid-template-columns: 1fr; }
  .hero { padding: 3rem 0 2rem; }
}
```

- [ ] **Step 3: Full-page verification pass**

Run: `open index.html`, then resize the window narrow (or DevTools device toolbar at 375px).
Expected, all true:
- Footer reads "© 2026" with three working links.
- At 375px: no horizontal page scroll; nav shows only the name + CV button; About stacks (photo above text); Research rows stack; project grid is one column; marquee still scrolls (or stills under reduced motion) without causing horizontal overflow.
- Sticky nav anchors jump correctly to About, Research, Projects.
- No leftover cog background, blue overlay, rainbow hover, or `#333` header anywhere.

- [ ] **Step 4: Validate HTML + confirm no stale references**

Run: `python3 -c "class P(__import__('html.parser',fromlist=['HTMLParser']).HTMLParser): pass; P().feed(open('index.html').read()); print('parsed-ok')"`
Run: `grep -n "floating-cogs\|rainbowShadow\|#333" index.html styles.css || echo "clean"`
Expected: `parsed-ok`, and `clean` (no references to the retired styles).

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css
git commit -m "Add footer and responsive layout; final polish"
```

- [ ] **Step 6: Push (only after Yann reviews the live-preview locally)**

```bash
git push
```

---

## Self-Review

**Spec coverage:**
- Editorial light theme, palette, fonts, retire cog/rainbow → Task 1 (+ enforced in Task 6 grep).
- Sticky nav + hero → Task 2.
- Expanded About (focus, education, interests, animated marquee, reduced-motion) → Task 3.
- Distinct Research section (GP Fusion + RAG) → Task 4.
- Projects restyled, grouped, all retained → Task 5.
- Footer year 2026, responsive, CV link intact → Tasks 2/6.
- Split into `index.html` + `styles.css`, no build step → Tasks 1–6.
- No invented content; About copy user-approved; interests requires Yann's text → Task 3 Step 1.

**Placeholder scan:** One deliberate call-out — Task 2 Step 2 contained an invalid `#symbol` token in a hover rule; the same step's note instructs replacing it with `filter: brightness(1.15)` and dropping the hover `background`. The interests line is intentionally deferred to Yann-provided text (a real content dependency, not a code placeholder), and Task 3 handles its absence explicitly.

**Type/name consistency:** Anchor ids (`#about`, `#research`, `#projects`, `#top`) are defined in Tasks 2–5 and referenced by the nav in Task 2. Class names (`.wrap`, `.eyebrow`, `.section-title`, `.marquee`, `.marquee-track`, `.research-item`, `.project-card`, `.nav-cv`) are each defined once in CSS and used consistently in markup. The marquee animation translates `-50%` and the track duplicates its tag list exactly, which is the invariant that makes the loop seamless.
