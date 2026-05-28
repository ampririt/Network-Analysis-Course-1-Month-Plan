# Network Analysis Course — Slide Design System

> Recipe for building the next session's deck so it feels like a continuation, not a different course.
> Source of truth: `Introduction_Session_1.html` (44 slides, single self-contained HTML).

---

## 1. The Big Idea

**"Network Lab"** — a quietly technical, paper-on-graph-grid aesthetic that respects the reader. Editorial serif for human warmth, monospace for technical exactness, a calm but committed color system. Density is **high (reading-first)** but every slide earns its weight with one visual or live animation; never a wall of bullets.

The deck is a single self-contained HTML file:
- 1920 × 1080 fixed-stage, scaled uniformly to the viewport (`viewport-base.css` pattern)
- All CSS and JS inline, no build step
- Fonts pulled from Google Fonts + Material Symbols CDN — no other external assets
- Inline editor available (press **E** or hover top-left), saves to `localStorage`

---

## 2. Color System (CSS variables)

```css
:root {
  /* Paper & ink */
  --paper:        #f4efe4;   /* page background */
  --paper-warm:   #ebe4d2;   /* card / panel fill */
  --ink:          #0f2030;   /* primary text + borders */
  --ink-soft:     #2a3a4a;   /* body text */
  --muted:        #6b7785;   /* secondary text */
  --rule:         #c9bfa8;   /* dividers, dotted lines */

  /* Four accent families — each owns a soft + bold pair */
  --cobalt:       #1d3fb8;   --cobalt-deep: #112879;   --cobalt-soft: #d6deff;
  --amber:        #d9621a;                              --amber-soft:  #f5d9bf;
  --moss:         #3a7d44;                              --moss-soft:   #cbe1ce;
  --plum:         #7a2057;                              --plum-soft:   #ecd5e2;

  /* Stage */
  --stage-bg:     #1a1612;   /* letterbox color outside the slide */
  --slide-bg:     var(--paper);
}
```

**Color semantics — keep these consistent across the course:**
- **Cobalt** = network / IP / routing / "what's happening on the wire"
- **Amber** = transport / encapsulation / packets in flight / things in motion
- **Moss** = address / identity / hardware (MAC, NIC) / success states
- **Plum** = data-link / framing / less-used edge cases
- The OSI 7-layer stack tier coloring: **Upper (L5-L7) = cobalt · Middle (L4) = amber · Lower (L1-L3) = moss**

Never use raw hex literals in new code — go through the variables so reskinning means changing one place.

---

## 3. Typography

```html
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,500;0,9..144,700;0,9..144,900;1,9..144,500;1,9..144,700&family=DM+Sans:wght@400;500;700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0&family=Material+Symbols+Rounded:opsz,wght,FILL,GRAD@24,500,1,0&display=swap" rel="stylesheet">
```

| Role | Family | Used for |
| --- | --- | --- |
| **Display** | `Fraunces` (serif, with italic) | h1, h2, h3, hero quotes, `em.display`. Italic Fraunces is the *signature* — use it for the one or two italicized words per title. |
| **Body** | `DM Sans` | Paragraphs, card descriptions, labels. |
| **Mono** | `JetBrains Mono` | Code, IP/MAC addresses, mastheads, eyebrows, page numbers, anything technical. |
| **Icons** | `Material Symbols Rounded` (filled) + `Outlined` (line) | Universal iconography. Use `.m-icon` (filled) by default, add `.outline` for the line variant. |

Default sizes — change only when there's a reason:
```css
--title-size:     168px;   /* title slide hero */
h2:               82px;    /* slide titles; shrink to 42–56 on dense slides */
h3:               38px;    /* card titles */
body p:           24px;    /* normal text */
.mono / tables:   14–22px;
.eyebrow:         18px mono, uppercase, 0.16em tracking
```

Wrap the one or two key concept words on each title in `<em class="display">…</em>` — that italic Fraunces moment is the deck's "voice".

---

## 4. Fixed Stage (NON-NEGOTIABLE)

Every slide is authored at exactly **1920 × 1080**. The whole stage scales uniformly to the viewport — letterboxed/pillarboxed, never reflowed. Phones still see 16:9.

Required HTML scaffold (the runtime handles scaling):

```html
<div class="deck-viewport">
  <main class="deck-stage" id="deckStage">
    <section class="slide active title-slide">…</section>
    <section class="slide">…</section>
    …
  </main>
</div>
```

`viewport-base.css` (paste verbatim — it's already in the file under "FIXED 16:9 STAGE: MANDATORY BASE STYLES") locks `.slide` to absolute 1920×1080, controls visibility via `.active`/`.visible`, and supports print + `prefers-reduced-motion`.

**Slide visibility is controlled by `.active`/`.visible` (visibility + opacity + pointer-events) — never `display:none/block`.** Several layout classes override `display`, and using `display` for switching breaks them.

---

## 5. Slide Chrome

Every non-title content slide gets the same masthead + page-number pattern. Page numbers are injected by JS at runtime from slide position (`renumber()` in `SlidePresentation`), so you don't have to hardcode them.

```html
<section class="slide">
  <div class="masthead">
    <div class="left"><span class="tag">SECTION</span><span>Slide topic · optional bilingual tag</span></div>
    <div class="right"></div>   <!-- JS fills with "07 / 44" -->
  </div>

  <span class="eyebrow reveal" style="--i:0;">Concept eyebrow · short, mono, uppercase</span>
  <h2 class="reveal" style="--i:1;">Main title — with one <em class="display">italic accent</em>.</h2>

  <!-- … slide-specific content … -->

  <div class="page-num"></div>     <!-- JS fills with "P. 07" -->
</section>
```

Graph-paper grid background + faint paper-grain overlay are applied by `.slide::before` / `.slide::after` — don't fight them.

The nav pill (`.deck-controls`) lives at top-right at **25% opacity** and brightens when the user moves the mouse. **Do not put content in the top-right 200×40px area or it will get covered.**

---

## 6. Reveal Animation System

```html
<element class="reveal" style="--i:N;">…</element>     <!-- fade + 24px slide up -->
<element class="reveal-left" style="--i:N;">…</element>  <!-- slide from left -->
<element class="reveal-scale" style="--i:N;">…</element> <!-- pop in -->
```

`--i:N` controls stagger via `transition-delay: calc(var(--i, 0) * 90ms + 100ms)`. Children of a slide reveal **only** when its container gets `.visible` (added by the SlidePresentation controller on `showSlide`). Use `--i` 0 → 7 within a slide.

---

## 7. Core Component Vocabulary

Each item exists in the current file — copy the pattern.

| Component | Purpose | Classes |
| --- | --- | --- |
| **Card** | Boxed bit of info with eyebrow + h3 + p. Color-tinted top stripe. | `.card` + tone (`.amber` / `.moss` / `.plum`) |
| **Pill** | Inline tag (technologies, OS support, …) | `.pill` |
| **Data table** | Spec/grading/protocol table with mono right-column | `table.data` |
| **Bit grid** | 32-bit IP visual, color-coded network/host | `.ip-bits .bit` (`.net` / `.host`) |
| **Header diagram** | Bit-accurate protocol header (Ethernet / IP / TCP) | `.hdr-diagram` + `.hdr-ruler` + `.field` |
| **OSI layer stack** | Animated 7-layer slide-in stack | `.osi-stack > .osi-layer` (tier: `.upper` / `.middle` / `.lower`) |
| **Layer deep-dive** | Per-layer slide template (big badge + lead + 3 info cards + footer + animation strip) | `.layer-hero` · `.layer-badge` (tier) · `.layer-info` · `.layer-viz` |
| **Layer animation strip** | One bite-sized CSS animation per layer | `.layer-viz` (auto-gets Start/Pause/Stop) |
| **Network panel** | LAN/WAN/Intra/Inter detailed card | `.net-panel` (tone) + `.net-illust` (pure-SVG illustration) |
| **Tool intro** | Wireshark / Packet Tracer style intro slide | `.tool-hero` + `.tool-logo` + `.tool-features` + `.tool-screenshot` |
| **Sim controls** | Big Start / Pause / Stop button row (interactive sims) | `.sim-controls` / `.ping-controls` / `.mail-controls` |
| **Generic anim controls** | Tiny `▶ ❚❚ ■` injected into every `[data-anim]` container | `.viz-controls` (auto-injected by JS) |

---

## 8. Iconography Rules

- **Material Symbols Rounded (filled)** is the default — `.m-icon` class with the icon's ligature name as the text:
  ```html
  <span class="m-icon">router</span>   <!-- renders the router icon -->
  ```
- Inside SVGs, use `<foreignObject>` to embed the Material icon span so coordinates stay aligned with lines/shapes:
  ```html
  <svg viewBox="0 0 700 140"><foreignObject x="…" y="…" width="…" height="…">
    <div xmlns="http://www.w3.org/1999/xhtml" class="il-node tier-tint">
      <span class="m-icon">router</span><span class="lbl">SWITCH</span>
    </div>
  </foreignObject></svg>
  ```
- **Never** rely on emoji for "icons that must align" — they sub-pixel-shift across platforms.
- Emoji is fine for the **office-mail analogy** (👔 📝 📦 🚚 — human-warmth metaphor) and similar narrative moments where alignment doesn't matter.
- Icons inherit color from `color:` on their parent — control tinting via `.lan-tint / .wan-tint / .intra-tint / .inter-tint` or set color directly.

---

## 9. Animation & Simulation Conventions

There are three tiers of animation in this deck — pick the right tier:

| Tier | When to use | How |
| --- | --- | --- |
| **Entrance reveal** | Every content slide. One staggered fade-in when the slide becomes `.visible`. | `.reveal` class + `--i:N` stagger |
| **Looping CSS visualization** | Layer animations (L1 wave, L4 handshake, …), bit streams in the wire, pulse dots on diagrams. Visual rhythm, no JS state. | CSS `@keyframes`, mark the container `data-anim` so it gets Start/Pause/Stop buttons. |
| **Interactive simulation** | Mail analogy, packet flow C→S, ping, protocol simulator, IP-vs-MAC hop traversal. Has logical state, narration, controls. | A custom class with `start() / pause() / stop()` methods. Register in `window._animOverrides[id]`. |

**Every animation must support Start / Pause / Stop.**
- For pure CSS loops: just add `data-anim` to the container. The generic controller in the JS section injects the buttons and toggles `.paused`/`.stopped` classes (which gate `animation-play-state` and `animation: none` respectively).
- For JS simulations: implement `start / pause / stop / reset` methods on a class, expose via `window._animOverrides[elementId] = instance`. The same `[data-anim]` mechanism will then call your methods instead of the CSS toggle.
- Interactive sims use **big visible controls** (`.sim-controls` row, Start = moss button, Pause = amber, Stop = paper). Generic loops use the **tiny corner pill** (`.viz-controls`).

Animation length guidelines:
- Reveal stagger: ~90 ms per `--i` step.
- Looping visual: 3–6 s/cycle so students can read what's happening.
- Interactive sim step: 1.7–2.0 s/step (long enough to read narration, short enough to stay engaging).

`prefers-reduced-motion` is honored globally (clamps all animations to 0.01 ms).

---

## 10. Slide Type Cookbook

The current deck uses about a dozen recurring layouts. When building Session 2+, pick one of these instead of inventing a new layout:

| Slide type | Pattern | Example in S1 |
| --- | --- | --- |
| **Title** | Big italic display, meta-row, 4-column spec strip, decorative "radar" rings | Slide 1 |
| **Section divider** | Marker mono line, huge italic h2, underline bar, italic blurb | Slide 9 (Session 1 divider), Slide 41 (takeaways) |
| **Course overview row** | 4-column card grid: 01/02/03/04 with tone tinting | Slide 2 |
| **Why-this-matters** | 3-column with big numeral + h3 + p | Slide 3 |
| **Tool intro** | `tool-hero` 2-col (logo left, lede + 4 feature cards right) + mock screenshot strip | Slides 32, 34 (Wireshark / Packet Tracer) |
| **Concept "what is X"** | Eyebrow + h2 + lead paragraph + 3- or 4-column card grid | Slides 11, 14 |
| **Layered diagram** | `.osi-stack` / `.collapse-wrap` / `.map-wrap` | Slides 15, 25, 24 |
| **Layer deep-dive** | `.layer-hero` template + per-layer animation in `.layer-viz` | Slides 17–23 |
| **Header anatomy** | `.hdr-diagram` 32-col bit grid with hoverable fields | Slides 28–30 |
| **Live simulation** | Side-by-side stage + `sim-controls` (Start/Pause/Stop) + narration | Slides 13 (mail), 27 (packet flow C→S), 39 (ping) |
| **Hands-on lab** | 2-col: numbered `.steps` left, support cards right | Slides 33 (Lab A), 35 (Lab B) |
| **Comparison split** | Two `.net-panel`s side by side + bottom `.net-compare` strip | Slides 7, 8 |
| **Take-aways closer** | Divider-style with 3 big numerals + tease for next session | Slide 44 |

---

## 11. JS Architecture

A single `<script>` block at the end of `<body>` holds:

1. **`class SlidePresentation`** — stage scaling, keyboard + touch + wheel nav, page renumbering, auto-fade nav pill. Created at top of the script.
2. **`class InlineEditor`** — toggle editing with E or hotzone, save to localStorage. Optional but included by default.
3. **Per-sim controllers** — IIFE pattern, each registers itself in `window._animOverrides[slideId]`:
   ```js
   (function() {
     const slide = document.getElementById('foo');
     if (!slide) return;
     // … set up state machine …
     window._animOverrides['foo'] = { start, pause, stop };
   })();
   ```
4. **Generic anim controller** — runs *last*, walks `document.querySelectorAll('[data-anim]')`, injects the corner-pill `.viz-controls`, calls the override if registered or toggles CSS classes otherwise.

When adding a new sim, follow that exact ordering.

---

## 12. Bilingual / Source-Attribution Convention

The deck draws content from a Japanese PPTX (`情報ネットワーク_第１回.pptx`) and translates it. Where a slide is sourced from the Japanese material, keep a small mono breadcrumb in the masthead's second-left span:

```html
<span>Network forms · ネットワークの形態</span>
```

Don't crowd the slide with the Japanese text — one short tag in the masthead is enough. The italic "Bottom line" / "Translated from PPTX p.X" line at the foot of a few slides is also OK for direct quotes.

---

## 13. Building the Next Session

Recipe to start `Session_2_Protocol_Stack.html`:

1. **Copy `Introduction_Session_1.html` as the starting template** — the CSS, JS, and `viewport-base.css` block are already wired up and battle-tested.
2. **Strip slides 8–44** (everything after the course overview) — keep slides 1–7 (title, course overview, sessions roadmap, tools, session shape) so the deck reads as a continuation.
3. **Update the title slide** — change "Vol. 01 — Foundations" → "Vol. 02 — Protocols", update the "Volume" spec to "II — Session 02", update the tagline.
4. **Add a fresh session divider** like slide 9 with the Session 2 topic.
5. **Build content using the cookbook in §10** — concept slides, header anatomies, deep dives, labs, takeaways. Re-use `.layer-hero`, `.hdr-diagram`, `.osi-stack`, etc.
6. **Per-layer / per-protocol animations** — add new ones following §9 (CSS loops with `data-anim` for visual rhythm, JS sims with explicit controllers for interactive walkthroughs).
7. **Keep the closing pair** — workshop slide + takeaways slide. Tease Session 3 in the takeaways.
8. **Open in a browser, walk every slide.** Verify no overflow, every animation Start/Pause/Stop works, page numbers reflow automatically.

### Don't break these invariants

- Fixed 1920×1080 stage, scaled uniformly — never reflow per device.
- `.slide` visibility via `.active`/`.visible` (visibility + opacity), never `display`.
- One italic display word per title — Fraunces italic is the voice.
- Color semantics from §2 (cobalt = network, amber = transport/motion, moss = identity/success, plum = data link).
- Every looping animation gets Start/Pause/Stop (via `data-anim` or its own controller).
- Page numbers auto-injected — leave `<div class="page-num"></div>` empty and `<div class="right"></div>` empty in mastheads.
- Inline editing stays on by default unless the slide is intentionally locked.

### Content order template for a session

1. Title (volume + session number)
2. "What we'll cover" overview
3. Session divider
4. Learning objectives (3–4 cards)
5. Lecture: vocabulary primer
6. Lecture: core concept slides with animations
7. Lecture: header-anatomy slides for any new protocols
8. **Tool intro** (`tool-hero` pattern) → then the matching **Lab** slide
9. Repeat tool-intro → lab for each new tool
10. Workshop activity + homework
11. Takeaways + next-session tease

---

## 14. File List

- `Introduction_Session_1.html` — the live Session-1 deck (44 slides, ~330 KB, single file, self-contained).
- `design.md` — this file. Update it whenever you change the design system.
- `情報ネットワーク_第１回.pptx` — Japanese source slides for Session 1.
- `ref_doc/` — textbooks and supplementary PDFs (Kurose-Ross, Wireshark Essentials, Packet Tracer labs).
- `README.md` — course-level plan covering all 8 sessions.
- `S1/` … `S8/` — per-session content drafts (readme + assets, structure varies).

When generating Session 2, the working file should be named `Session_2_<topic>.html` (e.g. `Session_2_Protocol_Stack.html`) and saved alongside this one.
