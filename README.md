# AURELIS HOROLOGY — Time, Refined.

An immersive, single-file cinematic website for a fictional independent luxury watchmaker. Built as a digital haute-horlogerie experience: scroll-driven storytelling, a procedural 3D watch, macro craftsmanship detail, and a live product configurator — all in one self-contained HTML file with no build step.

---

## 1. Quick Start

This is a **single HTML file** (`aurelis-horology.html`). There is no build process, no `npm install`, no bundler.

1. Download `aurelis-horology.html`.
2. Open it directly in a modern desktop browser (Chrome, Edge, Firefox, Safari) — double-click, or drag it into a browser window.
3. For the best experience, serve it from a local server instead of `file://` (some browsers throttle features like autoplay/audio or clipboard on `file://`):

   ```bash
   # any of these work from the folder containing the file
   npx serve .
   python3 -m http.server 8080
   ```

   Then visit `http://localhost:8080/aurelis-horology.html`.

No API keys, no environment variables, no dependencies to install locally — every library is pulled from a CDN at runtime (see §3).

---

## 2. What's Inside

| Section | What it does |
|---|---|
| Preloader | Full-black brand moment with a progress counter before the site becomes interactive |
| Hero | Procedural 3D watch (Three.js) with mouse parallax and scroll-driven camera inspection |
| Manifesto | Oversized scroll-revealed typography ("Every second has a mechanism") |
| Macro Inspection | Pinned section revealing finishing details via animated hotspots |
| Case Specs | Oversized numerals instead of spec-sheet tables |
| Movement / Exploded View | Scroll-scrubbed animation that explodes the movement into components, then reassembles it |
| Tourbillon | A continuously spinning cage with a short editorial explainer |
| Craftsmanship | Horizontal scroll-jacked gallery of the finishing process |
| Materials | Hover-tilt gallery of case/strap materials |
| Dial Collection | Selectable dial colors that update a live preview |
| Configurator | Fully wired case / dial / strap / buckle picker with live SVG preview and dynamic pricing |
| Collection | Horizontal scroll-jacked gallery of the four watch models |
| Passage of Time | A circular, clock-face scroll piece with rotating hand and typography |
| Finale | Closing cinematic moment before the footer |
| Footer | Standard site-wide links and legal |

Global features layered across all of the above: dark/light theme toggle ("Darkroom" / "Daylight"), a no-autoplay ambient sound toggle, a custom cursor with contextual labels, magnetic buttons, a fixed nav with live active-section tracking, a fullscreen mobile menu, and full `prefers-reduced-motion` support.

---

## 3. Tech Stack

Everything is loaded from CDN inside the single HTML file — nothing to install:

| Library | Purpose | Source |
|---|---|---|
| [Three.js r128](https://threejs.org/) | Procedural 3D watch in the hero (case, bezel, dial, hands, tourbillon cage) | cdnjs |
| [GSAP 3.12](https://gsap.com/) + ScrollTrigger | All scroll-scrubbed animation, pinning, and horizontal scroll sections | cdnjs |
| [Lenis](https://lenis.darkroom.engineering/) | Smooth inertia scrolling | cdnjs |
| Google Fonts — Fraunces + Space Grotesk | Editorial serif + geometric sans pairing | fonts.googleapis.com |

No React, no build tooling, no bundler — plain HTML/CSS/vanilla JS plus the above three runtime libraries.

> Since everything loads over `https://`, an internet connection is required the first time the page renders (fonts + libraries). If you need a fully offline copy, download the three CDN scripts and the font files locally and swap the `<script src>` / `<link>` URLs to relative paths.

---

## 4. Editing the Content

Because this is a single file, all copy and data live directly in the HTML/JS — there's no separate CMS or data folder. Here's where to look for common edits:

- **Copy & taglines** — search the HTML body for the section you want (e.g. `id="hero"`, `id="tourbillon"`); all text is inline.
- **Configurator options & pricing** — look for `CONFIG_DATA` and `priceMap` near the top of the `<script>` block. Case/dial/strap/buckle options and their SVG gradient IDs are defined there.
- **Watch colors/materials** — SVG gradients (`mg-case`, `case-rose-gold`, `dial-obsidian`, etc.) are defined in `<defs>` blocks inside the relevant `<svg>` elements (macro section, dial-select section, configurator section). Add a new gradient there and reference its `id` from a chip's `data-grad` attribute to add a new material/dial option.
- **Collection models** — the four `.coll-card` blocks inside `id="collection"` each hold a name, description, movement type, and price.
- **3D hero watch geometry** — inside the `initHeroScene()` function in the `<script>` block. Case, bezel, dial, hands, indices, tourbillon, and crown are each built from basic Three.js primitives (`CylinderGeometry`, `TorusGeometry`, `BoxGeometry`, etc.) so they're straightforward to resize or restyle by adjusting the constructor arguments and materials.

---

## 5. Known Limitations

- **No real 3D asset (GLB/GLTF)** — the hero watch is built procedurally from Three.js primitives rather than a modeled/scanned watch, so fine details (crown knurling, dial texture, hand shape) are simplified. Swapping in a real GLB model is possible by replacing the primitive-construction code in `initHeroScene()` with a `GLTFLoader` call.
- **No backend** — "Request Private Commission" and other CTAs are anchor links, not form submissions. Wire them to a real endpoint or form service if this goes to production.
- **Three.js r128** — pinned to an older version for stability; some newer PBR features (e.g. `transmission` on `MeshPhysicalMaterial`) are intentionally avoided because they render incorrectly on this build (see Troubleshooting below).
- **Fonts/libraries require network access** on first load since everything is CDN-hosted.
- **Desktop-first** — the 3D scene, horizontal scroll-jacking, and custom cursor are all tuned for and tested primarily on desktop viewports; mobile gets simplified interaction (no custom cursor, reduced motion sensitivity) but hasn't been exhaustively tested on every device.

---

## 6. Troubleshooting / Design Notes

A few non-obvious decisions worth knowing if you're extending this file:

- **Pinned sections must have `min-height` matching their scroll distance.** Sections that use `ScrollTrigger`'s `pin:true` (Macro, Exploded Movement, Passage of Time, Finale) intentionally use `min-height:100vh` on the outer `<section>`, matching the trigger's `end:'+=100%'`(or whatever distance is configured). If you extend a scroll animation's duration, increase the trigger's `end` value — **do not** just bump the section's CSS `min-height`, or you'll reintroduce a blank-scroll gap after the pin releases.
- **Don't combine CSS `position: sticky` with a GSAP `pin:true` on the same content.** Earlier versions of this file double-pinned some sections (CSS sticky on the inner wrapper *and* GSAP pin on the outer section), which caused a jarring pause/blank stretch. The inner wrappers in pinned sections use `position: relative`, not `sticky` — GSAP already handles keeping them fixed in the viewport during the pin.
- **SVG hand/needle elements need explicit `z-index`.** Where a rotating SVG needle sits behind foreground text (e.g. the Passage of Time section), the SVG and the text overlay both need explicit stacking (`z-index`) — relying on default DOM paint order isn't reliable once GSAP starts applying inline transforms to SVG children.
- **Three.js hand/pointer geometry should pivot from the mesh's local origin, not its bounding-box center.** Any mesh that needs to rotate like a clock hand should have its geometry translated so the pivot point sits at local `(0,0,0)`, then be positioned at the true rotation center and rotated in place — rotating a mesh that's merely *positioned* off-center will orbit it instead of pivoting it correctly.

---

## 7. Browser Support

Tested primarily on the latest Chrome/Edge (Chromium) and Firefox on desktop. Requires WebGL for the hero 3D scene; if WebGL is unavailable, the canvas is hidden and the hero gracefully falls back to typography over the atmospheric gradient background — the rest of the page is unaffected.

---

## 8. License / Usage

AURELIS is a fictional brand created for this demo. All copy, the "AURELIS" name, and visual identity are original. No real watch brand's assets, logos, or trademarks are used anywhere in this project.
