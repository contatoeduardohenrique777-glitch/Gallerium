# REBUILD COMMAND: "GALLERIUM" red-gradient hero with a giant stretch-reactive wordmark and a 3D ring of art cards

Build ONE file, `index.html`, that reproduces the page below exactly. It is a full-screen hero on a vertical gradient from red to soft pink. A giant white wordmark, "GALLERIUM", spans the full width under a thin header. Each of its nine letters has its own resting height, so the bottom edge is jagged. On mouse devices the letters stretch and shrink with the cursor's horizontal position. A Three.js ring of nine curved art cards spins slowly through the wordmark: the near half of the ring passes **in front** of the letters, and the far half shows **behind** them as dark-red silhouettes. At the bottom-left there is a two-line tagline. Every number in this command is measured. Do not interpret, restyle or "improve" anything.

**Assets: Path A (hosted on Cloudflare R2, CORS `*`).** The nine card images are already on a CDN. Use the URLs in section 5.3 exactly as written. Do not download, re-encode, rename or replace them, and never swap them for placeholders. Everything else is code: the wordmark and the icons are inline SVG (complete in APPENDIX A), the font comes from Google Fonts and Three.js comes from jsDelivr through an import map.

**Fastest correct path:** the complete, verified source is in APPENDIX A. Output it verbatim. Sections 0-12 explain every value so you can check your output. If anything in a section seems to conflict with the appendix, the appendix wins.

---

## 0. GLOBAL RULES (non-negotiable)

- One `index.html`, no build step, no framework, no bundler. Plain HTML + CSS + one `<script type="module">` (about 107 lines).
- The only library is **Three.js r176**, loaded with this exact import map (ES modules, no `<script src>`):
  ```
  "three": "https://cdn.jsdelivr.net/npm/three@0.176.0/build/three.module.js"
  "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.176.0/examples/jsm/"
  ```
  The ring uses the official `Flow` class from `three/addons/modifiers/CurveModifier.js`. Do not write your own bending shader, and do not use another version.
- `<html lang="en">`. All visible text, `aria-label`s and code comments are English.
- Font: Google Fonts, one `<link>` (plus two `preconnect` links):
  `https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@62..125,400&display=swap`
  Use it everywhere with `font-stretch: 118%; font-weight: 400; letter-spacing: -0.03em`.
- No scroll: `html, body {height:100%; margin:0}` and `body {overflow:hidden}`. The hero is `height:100svh` (fallback `100vh`).
- There is **no preloader**. The page starts in the "before" state of every entrance and the entrance plays as soon as the fonts are ready (section 10).

### 0.1 Tokens

| Token | Value | Use |
|---|---|---|
| `--bg` | `#E2231A` | gradient top (red) |
| `--bg-end` | `#E0A29F` | gradient bottom (soft pink) |
| hero background | `linear-gradient(180deg, var(--bg) 0%, var(--bg-end) 100%)` | top to bottom, full hero height; the header is transparent over it |
| `--ring-back` | `#A31A14` | far half of the ring (silhouettes behind the letters) |
| `--ink` | `#FFFFFF` | wordmark, all text, icons |
| `--ink-60` | `rgba(255,255,255,.6)` | header subtitle |
| `--line` | `rgba(255,255,255,.3)` | 1px header bottom border |
| card fallback | `#000000` | front card color until its image loads (or if it fails) |
| `--ease` | `cubic-bezier(.55,0,.1,1)` | every entrance transition |
| `--dur` | `1.5s` | every entrance transition |

## 1. FLUID SCALE (no fixed canvas)

The page is fluid, not a scaled canvas. Two CSS scale units drive every size:

- `--s: clamp(.5px, .06944vw, 1px)`: layout px (1px at widths >= 1440).
- `--t: clamp(.7px, calc(.4px + .041667vw), 1px)`: text px (1px at widths >= 1440).

Derived tokens (desktop, width > 667px):

| Token | Value | At 1440 × 900 |
|---|---|---|
| `--sp` (page gutter) | `20 * --s` | 20px |
| `--gx` (grid gutter) | `20 * --s` | 20px |
| `--hdr-h` | `60 * --s` | 60px |
| `--hdr-shift` | `-9 * --s` | -9px |
| `--top` (wordmark padding-top) | `80 * --s` | 80px |
| `--fz-small` / `--lh-small` | `13 * --t` / `16 * --t` | 13 / 16px |
| `--fz-body` / `--lh-body` | `26 * --t` / `30 * --t` | 26 / 30px |
| `--arrow` | `16 * --s` | 16px |
| `--nav-col` | `.5` (6/12); `5/12` when width <= 1199px or height <= 499px | 0.5 |

Big screens (`min-width:2400px and min-height:700px`): `--sp 30px; --hdr-h 80px; --top 110px; --fz-small 16px; --lh-small 20px; --fz-body 38px; --lh-body 45px; --arrow 24px`.

The wordmark is always `width: 100% - 2 * --sp`, so it fills ultrawide screens edge to edge. Never cap its width or center a fixed canvas.

## 1.5 COORDINATE TABLE (reference viewport 1440 × 900, final state after the entrance)

Text boxes are cap-trimmed (`text-box: trim-both cap alphabetic`), so `y` is the cap top and `h` is the cap height.

| Element | x | y | w | h | z | Notes |
|---|---|---|---|---|---|---|
| Header `.hdr` | 0 | 0 | 1440 | 60 | 10 (fixed) | transparent bg, 1px bottom border `--line` |
| Logo "GALLERIUM" | 20 | 11 | 85.8 | 8.9 | - | Archivo 13/16 |
| Subtitle "One Room. Every Artist" | 20 | 32.8 | 151.9 | 8.9 | - | `--ink-60` |
| Nav "About" (first) | 730 | 11 | 39.1 | 8.9 | - | 5 links, gap 40px |
| Nav "Login" | 1318.3 | 11 | 34.9 | 8.9 | - | right group, gap 40px |
| Nav "Join" (header) | 1393.2 | 11 | 26.8 | 8.9 | - | right edge 1420 |
| Back canvas `.gl--back` | 0 | 0 | 1440 | 900 | 0 | far half of the ring |
| Wordmark SVG | 20 | 76 | 1400 | 493 | 1 | viewBox 1420 × 500 |
| Front canvas `.gl--front` | 0 | 0 | 1440 | 900 | 3 | near half of the ring |
| Tagline `.foot p` (2 lines) | 20 | 832.2 | 417.9 | 47.8 | 4 | Archivo 26/30, last baseline at y 880 |

Wordmark letters at rest (1440 × 900, scaleY applied, `x, y, w, h`): G `20, 77.2, 152.2, 397.1` · A `186, 81.8, 163, 482` · L `362.8, 81.2, 124.6, 429.5` · L `501.2, 80.7, 124.6, 389.5` · E `639.6, 81.1, 126.9, 424` · R `780.3, 80, 150.1, 331.1` · I `944.2, 80.6, 59.3, 380.7` · U `1017.4, 81.2, 149.4, 431.5` · M `1180.6, 80.2, 239.5, 350.6`.

## 2. WORDMARK (PIXEL CONTRACT, the fragile point)

- One inline `<svg viewBox="0 0 1420 500">` with **nine `<path>` elements, one per letter, left to right**, each `fill="#fff"`, `data-char` and a `data-scale-y` resting value. The outlines are the Anton glyphs, already converted and fitted (cap height 494, y 6..500, letters 14 units apart, G at x 0 and M ending at x 1420). Copy the `d` attributes from APPENDIX A exactly.
- Resting `data-scale-y`, left to right: **G 0.800845 · A 0.989695 · L 0.881787 · L 0.799787 · E 0.870604 · R 0.679877 · I 0.78162 · U 0.877945 · M 0.719856**.
- Every path: `transform-origin: 50% 0` (top edge of the viewBox), `transform: scaleY(value)` set inline by JS. The letters hang from the top line and their bottoms end at different heights.
- The SVG is `display:block; width:100%; height:auto; overflow:visible` inside `.title`, which is `position:absolute; left:--sp; right:--sp; top: calc(--top - 4px)`. `overflow: visible` lets the G and U overshoots (y 1.51 and 504.49) show.
- DO NOT render the wordmark as live text, use a web font for it, add a stroke, or justify it with letter-spacing. It is nine SVG paths.
- DO NOT give all letters the same height at rest.

### 2.1 Cursor stretch (desktop only, mouse devices)

- `targetX = clientX / innerWidth` on every `mousemove` (window). `touchstart` resets `targetX = 0.5`. There is no reset on mouse leave.
- One smoothed value `smoothX` (starts at 0.5) is shared by the letters and the ring camera. Every frame: `dt = min(now - last, 60)` ms; `k = 0.05 * dt / 16`; if `|targetX - smoothX| < 1/2500` snap to the target, else `smoothX += (targetX - smoothX) * k`.
- For letter `i` (0 = G … 8 = M), `n = 9`, resting `d`:
  `dir = smoothX > .5 ? 1 : -1`, `amt = |smoothX - .5| * 2`, `g = 1 - 2i/n`
  `scale = clamp(d - g * dir * amt * min(1.5 - d, 0.2), 0.1, 1)`; when `amt = 0`, `scale = d`.
- Result: with the cursor at the left edge, G stretches (up to 1.0) and M shrinks by up to 0.2 × 7/9; at the right edge the opposite happens. No transition: the lerp is the only smoothing.
- Disabled on phones (≤ 667px) and when `prefers-reduced-motion: reduce`.
- DO NOT use CSS transitions or GSAP for this, and DO NOT move the letters horizontally.

## 3. CARD RING (PIXEL CONTRACT, Three.js)

- **Two transparent canvases** fill the hero (`position:absolute; inset:0; width:100%; height:100%`): `.gl--back` at z 0 (under the wordmark) and `.gl--front` at z 3 (over the wordmark). This split is what puts the near cards in front of the letters and the far cards behind them.
- Renderers: `new THREE.WebGLRenderer({canvas, antialias:true, alpha:true})`, `setPixelRatio(2)` (fixed 2, not devicePixelRatio), `toneMapping = NoToneMapping`, `setClearColor(0x000000, 0)`, `setSize(heroWidth, heroHeight, false)`. Default output color space (sRGB).
- Cameras: `PerspectiveCamera`. Back camera: `near 69, far 200`. Front camera: `near 0.1, far 69`. Both share:
  - `fov = 28 * (max(0, 1.8717948717948718 - w/h) * 0.5 + 1)` (31.8° at 1440 × 900, 35.5° at 1024 × 768, 47.7° at 375 × 812), `aspect = w/h`.
  - Desktop: `position (0, -14, -70)`, rotation (Euler XYZ) `x = -192°`, `y = 0`, `z = -25° + (smoothX - 0.5) * 0.2 rad`.
  - Phone (≤ 667px): `position (4, -17, -70)`, `x = -200°`, `z = -38° + (smoothX - 0.5) * 0.2 rad`.
- Curve: a circle of radius **15** in the XZ plane: `getPoint(t) = (cos(2πt)·15, 0, sin(2πt)·15)` (subclass of `THREE.Curve`).
- Cards: **9**. Each card is `new Flow(new THREE.Mesh(new THREE.PlaneGeometry(9, 12.6, 10, 1), material))`, then `flow.updateCurve(0, ring)` and `flow.moveAlongCurve(i / 9)`. Add `flow.object3D` to the scene. Each card exists twice: once in the front scene (its own material, textured with the card image) and once in the back scene (shared `--ring-back` material). Both copies get the same offsets, so they stay in sync.
- Materials: `MeshBasicMaterial`, `side: DoubleSide`. Front: starts `color 0x000000`; when its image loads it gets `map` and `color 0xffffff` (section 5.3). Back: `color #A31A14`, no texture.
- Spin: every frame, every flow `moveAlongCurve(dt * -5e-5)` with `dt = min(now - last, 60)` ms. That is one full turn every **20 s**. Pause rendering while the hero is out of view (IntersectionObserver). With reduced motion the ring is still, in its starting position.
- Render order per frame: back renderer (back scene, back camera), then front renderer (front scene, front camera).
- DO NOT use one canvas with a depth trick, OrbitControls, lights, shadows, rounded corners, borders or gaps other than the ones the geometry produces (9 × 9 units on a 94.25-unit circumference leaves about 1.47 units between cards).
- DO NOT crop, tint, round, frame or reorder the card images, and DO NOT add images to the back silhouettes.

## 4. BRAND (PIXEL CONTRACT)

- The header logo is the word **GALLERIUM** in Archivo 400 at `--fz-small`, white, no icon (the layout keeps a text-only logo). It is a link with `aria-label="GALLERIUM home"`.
- `<title>GALLERIUM | One Room. Every Artist</title>`.
- Favicon (inline data URI): an arch, like a gallery doorway, drawn as one even-odd path in `#E2231A`:
  ```html
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><path fill="#E2231A" fill-rule="evenodd" d="M5 29V14a11 11 0 0 1 22 0v15zM11 29V16a5 5 0 0 1 10 0v13z"/></svg>
  ```
- DO NOT redraw or restyle the icon, and DO NOT add outlines, gradients or a second color.

## 5. HEADER, TAGLINE, CARD IMAGES

### 5.1 Header (desktop)
- `position:fixed; top:0; left:0; width:100%; height:--hdr-h; z-index:10; background:none` (the hero gradient shows through), Archivo `--fz-small/--lh-small`. `::after` is a 1px bottom border in `--line`.
- `.hdr__row`: `position:absolute; inset:0 --sp; padding:--sp 0; translate:0 --hdr-shift`.
- Logo block at `left:0; top:--sp`. Subtitle `position:absolute; left:0; top: 21.8 * --t`, color `--ink-60`.
- Nav (5 links) at `left: (100% + --gx) * --nav-col`, `top:--sp`, `display:flex; gap: 2 * --sp`.
- Right group (Login, Join) at `right:0; top:--sp`, same gap.
- All header texts are cap-trimmed (`text-box: trim-both cap alphabetic`) and `white-space:nowrap`.
- Phone: height 40px, only the logo (`top:11.5px`, 12px) and a burger button on the right (40 × 40, two 40 × 1px white bars at top 17px and 22px). The border is always visible on phone.

### 5.2 Tagline
- `.foot`: `position:absolute; z-index:4; left:--sp; bottom:--sp`. The `<p>` is cap-trimmed, `--fz-body/--lh-body`, letter-spacing -0.03em.
- Desktop, 2 lines, left-aligned: `One Gallery.` / `Show it. Get seen. Get collected`, followed by a 16px hooked arrow (`.foot__arrow--d`) at `right:-1.05em; top:.1em`.
- Phone, 3 lines, right-aligned, `bottom: --sp` (last baseline at y 792 on 375 × 812): `One Gallery.` / `Show it. Get seen.` / `Get collected`, with the same arrow, mirrored (`rotateY(180deg)`), 24px wide, at `left:-1.6em; top:-.1em`.

### 5.3 Card images (Path A, R2)

`CARD_IMAGES` holds these 9 URLs in this order (index 0 = first card, ring offset 0/9). In the appendix they are built from `ASSET_BASE = 'https://assets.layerweb.com.br/gallerium/images/'`. Each is a 750 × 1050 WebP (aspect 9:12.6, the card aspect), already cropped:

| # | URL |
|---|---|
| 1 | `https://assets.layerweb.com.br/gallerium/images/card-01.webp` |
| 2 | `https://assets.layerweb.com.br/gallerium/images/card-02.webp` |
| 3 | `https://assets.layerweb.com.br/gallerium/images/card-03.webp` |
| 4 | `https://assets.layerweb.com.br/gallerium/images/card-04.webp` |
| 5 | `https://assets.layerweb.com.br/gallerium/images/card-05.webp` |
| 6 | `https://assets.layerweb.com.br/gallerium/images/card-06.webp` |
| 7 | `https://assets.layerweb.com.br/gallerium/images/card-07.webp` |
| 8 | `https://assets.layerweb.com.br/gallerium/images/card-08.webp` |
| 9 | `https://assets.layerweb.com.br/gallerium/images/card-09.webp` |

- Load each with `THREE.TextureLoader` (anonymous CORS; the bucket answers `Access-Control-Allow-Origin: *`), set `tex.colorSpace = THREE.SRGBColorSpace`, then `map = tex`, `color = 0xffffff`, `needsUpdate = true`. Until a texture arrives (or if it fails), that card stays solid black.
- `Flow` clones the mesh AND its material. The texture must go on `front.object3D.material` (the clone that renders), never on the material passed to `new THREE.Mesh(...)`, which is never drawn. The appendix does this with `const cardMat = front.object3D.material`.
- Seen from the camera the images are upright and not mirrored with the default texture settings. DO NOT flip UVs or change `tex.flipY`.

## 6. LITERAL TEXTS (English, exact)

| Place | Text |
|---|---|
| `<title>` | `GALLERIUM \| One Room. Every Artist` |
| `<h1 class="sr-only">` | `GALLERIUM One Room. Every Artist` |
| Logo / subtitle | `GALLERIUM` / `One Room. Every Artist` |
| Nav | `About`, `Exhibitions`, `Artist Directory`, `Pricing`, `FAQ` |
| Right group | `Login`, `Join` |
| Tagline | `One Gallery.` / `Show it. Get seen.` / `Get collected` |
| Burger | `aria-label="Open menu"` |

## 7. LAYERING (bottom to top)

hero gradient → `.gl--back` (z 0) → `.title` (z 1) → `.gl--front` (z 3) → `.foot` (z 4) → `.hdr` (fixed, z 10, transparent). The hero has `isolation:isolate; overflow:hidden`, which also clips the entrance offsets.

## 8. RESPONSIVE (explicit architectures)

### 8.1 Desktop and tablet (width > 667px)
Everything in sections 1-5 with the fluid scale. Checked at 2560 × 1080, 1920 × 1080, 1440 × 900, 1024 × 768 and 768 × 1024. At 2560 × 1080 the wordmark is 2500px wide and runs below the fold behind the tagline. This is intended (the wordmark always fills the width).

### 8.2 Phone (width ≤ 667px, checked at 375 × 812)
- Tokens: `--s 1px; --t 1px; --sp 20px; --gx 10px; --hdr-h 40px; --top 60px; --fz-small 12px; --fz-body 21px; --lh-body 25px; --arrow 24px`.
- The desktop SVG is hidden and `.title__mobile` is shown: viewBox `0 0 351 312`, `preserveAspectRatio="none"`, one path with two rows, `GALLE` (y 2..148, full width) and `RIUM` (y 166..312, right-aligned). Width 335px, height `min(402px, max(312px, 48.75svh))`, or `51.5385svh` when the viewport height is ≥ 730px. At 375 × 812 it is 335 × 402 at (20, 60). No cursor stretch on phone.
- Ring camera and tagline use the phone values given above. Measured at 375 × 812: tagline box (20, 727.6, 335 × 64.4); logo cap top at y 11.5.

## 9. ENTRANCE TIMELINE (no preloader)

Start: `Promise.race([document.fonts.ready, 1500 ms timeout])`, then two `requestAnimationFrame`s, then add `is-in` to `<html>`. Every move is a CSS `transition` of `1.5s cubic-bezier(.55,0,.1,1)`:

| Element | From | To | Delay |
|---|---|---|---|
| Header | `opacity:0` | `opacity:1` | 0 |
| Header border (`::after`) | `scaleX(0)` (desktop) | `scaleX(1)` | 0 |
| Wordmark SVG | `translateY(-80%) scaleY(0)` | `translateY(0) scaleY(1)` | 0 |
| Tagline `.foot` | `translateY(120%)` | `translateY(0)` | 0 |
| Both canvases | `translateY(100%) rotate(35deg)` (phone `translate(-100%,50%) rotate(35deg)`) | `none` | 1s |

The ring spins and the cursor stretch runs from the first frame (they are simply off-screen until the canvases arrive). Total entrance: 2.5s.

## 10. ACCESSIBILITY

- `prefers-reduced-motion: reduce`: no transitions (everything jumps to its final state the moment `is-in` is added, after the fonts load or 1.5 s at most), no ring spin, and the `mousemove` listener is never registered, so `smoothX` stays 0.5: no cursor stretch and no camera tilt.
- One `<h1 class="sr-only">` carries the readable title. The wordmark, the canvases and the icons are `aria-hidden`.
- The nav has `aria-label="Main navigation"`; the burger and the logo have `aria-label`s. The card images are WebGL textures inside `aria-hidden` canvases (decorative).

## 11. GLOBAL "DO NOT" LIST

- DO NOT add a preloader, a loading screen or a fade over the whole page.
- DO NOT change the colors, the font, the sizes, the coordinates or the texts.
- DO NOT add a brush stroke, a fixed Join button or any other decoration. This hero has only the header, the wordmark, the ring and the tagline.
- DO NOT replace the card images, and DO NOT use a solid background instead of the red-to-pink gradient.
- DO NOT use one canvas for the ring, a different Three.js version, or a custom bending shader instead of `Flow`.
- DO NOT render the wordmark as text or give its letters equal heights.
- DO NOT add scroll, hover effects, a custom cursor, GSAP, Lenis or any other library.
- DO NOT set the renderer pixel ratio to `devicePixelRatio`; it is fixed at 2.

## 12. SELF-CHECK (run before answering)

1. At 1440 × 900 the wordmark SVG is 1400 × 493 at (20, 76) and the G path box is about (20, 77.2, 152.2 × 397.1).
2. The nine letter bottoms are uneven at rest (A is the longest, R the shortest).
3. Move the mouse to the left edge: G grows to full height and M shrinks; move it to the right edge: the opposite. The ring tilts slightly with the cursor.
4. The ring shows the 9 card images (upright, not mirrored), turns once every 20 s, the near cards cover the letters, and dark-red `#A31A14` card silhouettes show behind the letters.
5. The header "About" starts at x 730 and "Join" ends at x 1420 (1440 wide). The background runs from `#E2231A` at the top edge to `#E0A29F` at the bottom edge. Nothing sits in the bottom-right corner.
6. At 375 × 812: the title reads GALLE / RIUM at 335 × 402, and the tagline is right-aligned on three lines with its last baseline 20px above the bottom edge.
7. The entrance plays once in about 2.5s with no preloader. With reduced motion everything is static and final.
8. Archivo loads (`document.fonts.check('400 13px Archivo')` is true), Three.js, `CurveModifier.js` and the 9 card images load with status 200, and the console has no errors.

---

## APPENDIX A: complete `index.html` (output verbatim)

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>GALLERIUM | One Room. Every Artist</title>
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Cpath fill='%23E2231A' fill-rule='evenodd' d='M5 29V14a11 11 0 0 1 22 0v15zM11 29V16a5 5 0 0 1 10 0v13z'/%3E%3C/svg%3E">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@62..125,400&display=swap" rel="stylesheet">
<style>
  /* ---------- scale + tokens ---------- */
  :root {
    --bg: #E2231A;            /* page red (gradient top) */
    --bg-end: #E0A29F;        /* gradient bottom */
    --ring-back: #A31A14;     /* far half of the card ring */
    --ink: #FFFFFF;
    --ink-60: rgba(255, 255, 255, .6);
    --line: rgba(255, 255, 255, .3);
    --s: clamp(.5px, .06944vw, 1px);                    /* layout px */
    --t: clamp(.7px, calc(.4px + .041667vw), 1px);      /* text px */
    --sp: calc(var(--s) * 20);                          /* page gutter */
    --gx: calc(var(--s) * 20);                          /* grid gutter */
    --hdr-h: calc(var(--s) * 60);
    --hdr-shift: calc(var(--s) * -9);
    --top: calc(var(--s) * 80);                         /* title padding-top */
    --fz-small: calc(var(--t) * 13);
    --lh-small: calc(var(--t) * 16);
    --fz-body: calc(var(--t) * 26);
    --lh-body: calc(var(--t) * 30);
    --arrow: calc(var(--s) * 16);
    --nav-col: .5;                                      /* logo column = 6/12 */
    --ease: cubic-bezier(.55, 0, .1, 1);
    --dur: 1.5s;
  }
  @media (max-width: 1199px), (max-height: 499px) { :root { --nav-col: calc(5 / 12); } }
  @media (min-width: 2400px) and (min-height: 700px) {
    :root {
      --sp: 30px; --hdr-h: 80px; --top: 110px;
      --fz-small: 16px; --lh-small: 20px; --fz-body: 38px; --lh-body: 45px;
      --arrow: 24px;
    }
  }

  /* ---------- shell ---------- */
  *, *::before, *::after { box-sizing: border-box; }
  html, body { margin: 0; height: 100%; }
  body {
    background: var(--bg); color: var(--ink); overflow: hidden;
    font-family: 'Archivo', Helvetica, Arial, sans-serif; font-stretch: 118%; font-weight: 400;
    letter-spacing: -.03em; -webkit-font-smoothing: antialiased;
  }
  a { color: inherit; text-decoration: none; }
  .sr-only { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }
  .trim { text-box: trim-both cap alphabetic; }

  /* ---------- header ---------- */
  .hdr {
    position: fixed; top: 0; left: 0; width: 100%; height: var(--hdr-h); z-index: 10;
    font-size: var(--fz-small); line-height: var(--lh-small);
    opacity: 0; transition: opacity var(--dur) var(--ease);
  }
  .hdr::after {
    content: ""; position: absolute; left: 0; bottom: 0; width: 100%; border-bottom: 1px solid var(--line);
    transform: scaleX(0); transition: transform var(--dur) var(--ease);
  }
  .hdr__row { position: absolute; inset: 0 var(--sp); padding: var(--sp) 0; translate: 0 var(--hdr-shift); }
  .hdr__logo { position: absolute; left: 0; top: var(--sp); }
  .hdr__logo a { display: block; }
  .hdr__logo-sub { position: absolute; left: 0; top: calc(var(--t) * 21.8); color: var(--ink-60); white-space: nowrap; }
  .hdr__nav { position: absolute; top: var(--sp); left: calc((100% + var(--gx)) * var(--nav-col)); display: flex; gap: calc(var(--sp) * 2); }
  .hdr__right { position: absolute; top: var(--sp); right: 0; display: flex; gap: calc(var(--sp) * 2); }
  .hdr a, .hdr p { margin: 0; white-space: nowrap; }
  .hdr__burger { display: none; }

  /* ---------- hero ---------- */
  .hero { position: relative; height: 100vh; height: 100svh; overflow: hidden; isolation: isolate;
         background: linear-gradient(180deg, var(--bg) 0%, var(--bg-end) 100%); }
  .gl { position: absolute; inset: 0; width: 100%; height: 100%; display: block;
        transform: translateY(100%) rotate(35deg); transition: transform var(--dur) var(--ease) 1s; }
  .gl--back { z-index: 0; }
  .gl--front { z-index: 3; pointer-events: none; }
  .title { position: absolute; z-index: 1; left: var(--sp); right: var(--sp); top: calc(var(--top) - 4px); }
  .title svg { display: block; width: 100%; height: auto; overflow: visible;
               transform: translateY(-80%) scaleY(0); transition: transform var(--dur) var(--ease); }
  .title path { transform-origin: 50% 0; }
  .title__mobile { display: none !important; }
  .foot { position: absolute; z-index: 4; left: var(--sp); bottom: var(--sp);
          transform: translateY(120%); transition: transform var(--dur) var(--ease); }
  .foot p { margin: 0; font-size: var(--fz-body); line-height: var(--lh-body); letter-spacing: -.03em; }
  .foot__last { position: relative; white-space: nowrap; }
  .foot__arrow { position: absolute; top: .1em; right: -1.05em; width: var(--arrow); height: 100%; }
  .foot__arrow--m { display: none; }
  .foot__br-m { display: none; }

  /* ---------- entrance (class added on load) ---------- */
  .is-in .hdr { opacity: 1; }
  .is-in .hdr::after { transform: scaleX(1); }
  .is-in .title svg { transform: translateY(0) scaleY(1); }
  .is-in .gl { transform: none; }
  .is-in .foot { transform: translateY(0); }

  /* ---------- phone ---------- */
  @media (max-width: 667px) {
    :root {
      --s: 1px; --t: 1px; --sp: 20px; --gx: 10px; --hdr-h: 40px; --top: 60px;
      --fz-small: 12px; --lh-small: 16px; --fz-body: 21px; --lh-body: 25px;
      --arrow: 24px;
    }
    .hdr::after { transform: none; }
    .hdr__row { translate: none; padding: 0; }
    .hdr__logo { top: 11.5px; }
    .hdr__logo-sub, .hdr__nav, .hdr__right { display: none; }
    .hdr__burger { display: block; position: absolute; right: 0; top: 0; width: 40px; height: 40px; padding: 0; border: 0; background: none; cursor: pointer; }
    .hdr__burger::before, .hdr__burger::after { content: ""; position: absolute; left: 0; width: 40px; height: 1px; background: var(--ink); }
    .hdr__burger::before { top: 17px; }
    .hdr__burger::after { top: 22px; }
    .title { top: var(--top); }
    .title__desktop { display: none !important; }
    .title__mobile { display: block !important; height: min(402px, max(312px, 48.75svh)) !important; }
    .gl { transform: translate(-100%, 50%) rotate(35deg); }
    .foot { left: var(--sp); right: var(--sp); bottom: var(--sp); text-align: right; }
    .foot__br-m { display: inline; }
    .foot__arrow--d { display: none; }
    .foot__arrow--m { display: block; right: auto; left: -1.6em; top: -.1em; transform: rotateY(180deg); }
  }
  @media (max-width: 667px) and (min-height: 730px) {
    .title__mobile { height: min(402px, max(312px, 51.5385svh)) !important; }
  }

  /* ---------- reduced motion ---------- */
  @media (prefers-reduced-motion: reduce) {
    .hdr, .hdr::after, .gl, .title svg, .foot { transition: none !important; }
  }
</style>
<script type="importmap">
{ "imports": {
  "three": "https://cdn.jsdelivr.net/npm/three@0.176.0/build/three.module.js",
  "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.176.0/examples/jsm/"
} }
</script>
</head>
<body>

<header class="hdr">
  <div class="hdr__row">
    <div class="hdr__logo">
      <a href="#" class="trim" aria-label="GALLERIUM home">GALLERIUM</a>
      <p class="hdr__logo-sub trim">One Room. Every Artist</p>
    </div>
    <nav class="hdr__nav" aria-label="Main navigation">
      <a href="#" class="trim">About</a>
      <a href="#" class="trim">Exhibitions</a>
      <a href="#" class="trim">Artist Directory</a>
      <a href="#" class="trim">Pricing</a>
      <a href="#" class="trim">FAQ</a>
    </nav>
    <div class="hdr__right">
      <a href="#" class="trim">Login</a>
      <a href="#" class="trim">Join</a>
    </div>
    <button class="hdr__burger" type="button" aria-label="Open menu"></button>
  </div>
</header>

<main class="hero" id="hero">
  <h1 class="sr-only">GALLERIUM One Room. Every Artist</h1>
  <canvas class="gl gl--back" aria-hidden="true"></canvas>

  <div class="title" aria-hidden="true">
    <!-- desktop wordmark: 9 glyphs, left to right, each with its resting vertical scale -->
    <svg class="title__desktop" viewBox="0 0 1420 500" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path data-char data-scale-y="0.800845" fill="#fff" d="M66.74 504.49L66.74 504.49Q0 504.49 0 352.92L0 352.92L0 139.89Q0 1.51 76.47 1.51L76.47 1.51Q107.98 1.51 124.8 21.58Q141.62 41.65 147.99 80.1Q154.36 118.55 154.36 173.85L154.36 173.85L93.47 173.85L93.47 130.06Q93.47 114.06 90.72 103.26Q87.98 92.45 79.3 92.45L79.3 92.45Q68.51 92.45 65.05 103.96Q61.6 115.47 61.6 129.22L61.6 129.22L61.6 368.64Q61.6 388.57 64.88 401.06Q68.15 413.55 77.71 413.55L77.71 413.55Q87.62 413.55 90.9 401.06Q94.17 388.57 94.17 368.08L94.17 368.08L94.17 297.63L77.53 297.63L77.53 212.3L153.65 212.3L153.65 500L128.69 500L118.07 458.46Q101.79 504.49 66.74 504.49Z"/>
      <path data-char data-scale-y="0.989695" fill="#fff" d="M228.9 500L168.36 500L198.45 6L304.14 6L333.7 500L274.75 500L270.32 420.29L232.8 420.29L228.9 500ZM249.08 90.2L237.22 341.41L265.55 341.41L251.91 90.2L249.08 90.2Z"/>
      <path data-char data-scale-y="0.881787" fill="#fff" d="M474.09 500L347.7 500L347.7 6L410.01 6L410.01 415.8L474.09 415.8L474.09 500Z"/>
      <path data-char data-scale-y="0.799787" fill="#fff" d="M614.48 500L488.09 500L488.09 6L550.4 6L550.4 415.8L614.48 415.8L614.48 500Z"/>
      <path data-char data-scale-y="0.870604" fill="#fff" d="M757.18 500L628.48 500L628.48 6L753.1 6L753.1 101.43L691.86 101.43L691.86 197.71L750.63 197.71L750.63 290.61L691.86 290.61L691.86 403.73L757.18 403.73L757.18 500Z"/>
      <path data-char data-scale-y="0.679877" fill="#fff" d="M834.19 500L771.18 500L771.18 6L866.77 6Q890.66 6 902.88 23.26Q915.09 40.52 919.25 71.54Q923.41 102.55 923.41 144.38L923.41 144.38Q923.41 184.79 916.95 208.93Q910.49 233.07 892.61 242.33L892.61 242.33Q907.3 247.11 913.23 265.49Q919.16 283.88 919.16 313.07L919.16 313.07L919.16 500L857.74 500L857.74 306.61Q857.74 285 852.16 279.81Q846.59 274.61 834.19 274.61L834.19 274.61L834.19 500ZM834.55 97.5L834.55 189L849.6 189Q862.52 189 862.52 144.38L862.52 144.38Q862.52 115.47 859.69 106.48Q856.85 97.5 849.06 97.5L849.06 97.5L834.55 97.5Z"/>
      <path data-char data-scale-y="0.78162" fill="#fff" d="M997.6 500L937.41 500L937.41 6L997.6 6L997.6 500Z"/>
      <path data-char data-scale-y="0.877945" fill="#fff" d="M1087.36 504.49L1087.36 504.49Q1048.42 504.49 1030.01 469.83Q1011.6 435.16 1011.6 367.8L1011.6 367.8L1011.6 6L1072.14 6L1072.14 363.87Q1072.14 376.22 1073.03 387.59Q1073.91 398.95 1077.1 406.25Q1080.28 413.55 1087.36 413.55L1087.36 413.55Q1094.62 413.55 1097.81 406.39Q1100.99 399.24 1101.79 387.73Q1102.59 376.22 1102.59 363.87L1102.59 363.87L1102.59 6L1163.13 6L1163.13 367.8Q1163.13 435.16 1144.72 469.83Q1126.31 504.49 1087.36 504.49Z"/>
      <path data-char data-scale-y="0.719856" fill="#fff" d="M1233.42 500L1177.13 500L1177.13 6L1272.01 6L1298.21 307.17L1324.23 6L1420 6L1420 500L1363 500L1363 144.1L1327.24 500L1271.3 500L1233.42 144.1L1233.42 500Z"/>
    </svg>
    <!-- phone wordmark: GALLE / RIUM, stretched to the box height -->
    <svg class="title__mobile" viewBox="0 0 351 312" preserveAspectRatio="none" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path fill="#fff" d="M31.88 149.33L31.88 149.33Q0 149.33 0 104.53L0 104.53L0 41.57Q0 0.67 36.54 0.67L36.54 0.67Q51.59 0.67 59.63 6.6Q67.66 12.54 70.7 23.9Q73.75 35.26 73.75 51.61L73.75 51.61L44.66 51.61L44.66 38.67Q44.66 33.94 43.34 30.74Q42.03 27.55 37.89 27.55L37.89 27.55Q32.73 27.55 31.08 30.95Q29.43 34.35 29.43 38.42L29.43 38.42L29.43 109.18Q29.43 115.07 31 118.76Q32.56 122.45 37.13 122.45L37.13 122.45Q41.86 122.45 43.43 118.76Q44.99 115.07 44.99 109.01L44.99 109.01L44.99 88.19L37.04 88.19L37.04 62.97L73.41 62.97L73.41 148L61.49 148L56.41 135.72Q48.63 149.33 31.88 149.33ZM106.67 148L77.75 148L92.13 2L142.62 2L156.74 148L128.58 148L126.46 124.44L108.53 124.44L106.67 148ZM116.32 26.89L110.65 101.13L124.18 101.13L117.67 26.89L116.32 26.89ZM221.13 148L160.74 148L160.74 2L190.51 2L190.51 123.11L221.13 123.11L221.13 148ZM285.51 148L225.13 148L225.13 2L254.9 2L254.9 123.11L285.51 123.11L285.51 148ZM351 148L289.51 148L289.51 2L349.05 2L349.05 30.2L319.79 30.2L319.79 58.66L347.87 58.66L347.87 86.12L319.79 86.12L319.79 119.55L351 119.55L351 148ZM79.19 312L49.08 312L49.08 166L94.75 166Q106.17 166 112 171.1Q117.84 176.2 119.82 185.37Q121.81 194.54 121.81 206.9L121.81 206.9Q121.81 218.84 118.73 225.98Q115.64 233.11 107.1 235.85L107.1 235.85Q114.12 237.26 116.95 242.69Q119.78 248.13 119.78 256.75L119.78 256.75L119.78 312L90.44 312L90.44 254.84Q90.44 248.46 87.77 246.92Q85.11 245.39 79.19 245.39L79.19 245.39L79.19 312ZM79.36 193.04L79.36 220.09L86.54 220.09Q92.72 220.09 92.72 206.9L92.72 206.9Q92.72 198.35 91.37 195.7Q90.01 193.04 86.29 193.04L86.29 193.04L79.36 193.04ZM154.57 312L125.81 312L125.81 166L154.57 166L154.57 312ZM194.77 313.33L194.77 313.33Q176.16 313.33 167.36 303.08Q158.57 292.84 158.57 272.93L158.57 272.93L158.57 166L187.49 166L187.49 271.77Q187.49 275.42 187.92 278.78Q188.34 282.14 189.86 284.29Q191.38 286.45 194.77 286.45L194.77 286.45Q198.23 286.45 199.76 284.33Q201.28 282.22 201.66 278.82Q202.04 275.42 202.04 271.77L202.04 271.77L202.04 166L230.96 166L230.96 272.93Q230.96 292.84 222.17 303.08Q213.37 313.33 194.77 313.33ZM261.86 312L234.96 312L234.96 166L280.3 166L292.81 255.01L305.25 166L351 166L351 312L323.77 312L323.77 206.81L306.68 312L279.96 312L261.86 206.81L261.86 312Z"/>
    </svg>
  </div>

  <canvas class="gl gl--front" aria-hidden="true"></canvas>

  <div class="foot">
    <p class="trim">One Gallery. <br>Show it. Get seen. <br class="foot__br-m"><span class="foot__last"><svg class="foot__arrow foot__arrow--m" viewBox="0 0 14 14" aria-hidden="true"><path d="M2 3c0 5 3 8 9 8M7 7l4 4-4 4" fill="none" stroke="#fff" stroke-width="1.2"/></svg>Get collected<svg class="foot__arrow foot__arrow--d" viewBox="0 0 14 14" aria-hidden="true"><path d="M2 3c0 5 3 8 9 8M7 7l4 4-4 4" fill="none" stroke="#fff" stroke-width="1.2"/></svg></span></p>
  </div>
</main>

<script type="module">
import * as THREE from 'three';
import { Flow } from 'three/addons/modifiers/CurveModifier.js';

// Card art: portrait 750x1050 WebP (aspect 9:12.6). null = solid black card.
const ASSET_BASE = 'https://assets.layerweb.com.br/gallerium/images/';
const CARD_IMAGES = [1, 2, 3, 4, 5, 6, 7, 8, 9].map(n => `${ASSET_BASE}card-0${n}.webp`);

const hero = document.getElementById('hero');
const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
const phone = matchMedia('(max-width: 667px)');

/* ---------- entrance ---------- */
const start = () => requestAnimationFrame(() => requestAnimationFrame(() => document.documentElement.classList.add('is-in')));
Promise.race([document.fonts.ready, new Promise(r => setTimeout(r, 1500))]).then(start);

/* ---------- shared smoothed pointer x (0..1) ---------- */
let targetX = 0.5, smoothX = 0.5;
if (!reduce) addEventListener('mousemove', e => { targetX = e.clientX / innerWidth; });
addEventListener('touchstart', () => { targetX = 0.5; }, { passive: true });

/* ---------- letter stretch ---------- */
const chars = [...document.querySelectorAll('.title__desktop [data-char]')];
const base = chars.map(p => Number(p.dataset.scaleY) || 1);
chars.forEach((p, i) => { p.style.transform = `scaleY(${base[i]})`; });
function updateLetters(x) {
  if (phone.matches) return;
  const dir = x > 0.5 ? 1 : -1, amt = Math.abs(x - 0.5) * 2, n = chars.length;
  chars.forEach((p, i) => {
    const d = base[i], g = 1 - (2 * i) / n;
    let s = d;
    if (amt > 0) s = Math.max(0.1, Math.min(1, d - g * dir * amt * Math.min(1.5 - d, 0.2)));
    p.style.transform = `scaleY(${s})`;
  });
}

/* ---------- card ring (two renderers split at depth 69) ---------- */
const backCanvas = document.querySelector('.gl--back');
const frontCanvas = document.querySelector('.gl--front');
const makeRenderer = canvas => {
  const r = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
  r.setPixelRatio(2);
  r.toneMapping = THREE.NoToneMapping;
  r.setClearColor(0x000000, 0);
  return r;
};
const backR = makeRenderer(backCanvas), frontR = makeRenderer(frontCanvas);
const backCam = new THREE.PerspectiveCamera(28, 1, 69, 200);
const frontCam = new THREE.PerspectiveCamera(28, 1, 0.1, 69);
const backScene = new THREE.Scene(), frontScene = new THREE.Scene();

class Ring extends THREE.Curve {
  getPoint(t, target = new THREE.Vector3()) {
    return target.set(Math.cos(t * Math.PI * 2) * 15, 0, Math.sin(t * Math.PI * 2) * 15);
  }
}
const ring = new Ring();
const backMat = new THREE.MeshBasicMaterial({ color: new THREE.Color(getComputedStyle(document.documentElement).getPropertyValue('--ring-back').trim()), side: THREE.DoubleSide });
const loader = new THREE.TextureLoader();
const flows = [];
CARD_IMAGES.forEach((url, i) => {
  const mat = new THREE.MeshBasicMaterial({ color: 0x000000, side: THREE.DoubleSide });
  const front = new Flow(new THREE.Mesh(new THREE.PlaneGeometry(9, 12.6, 10, 1), mat));
  const back = new Flow(new THREE.Mesh(new THREE.PlaneGeometry(9, 12.6, 10, 1), backMat));
  // Flow clones the material, so the texture goes on the clone it actually renders
  const cardMat = front.object3D.material;
  if (url) loader.load(url, tex => { tex.colorSpace = THREE.SRGBColorSpace; cardMat.map = tex; cardMat.color.set(0xffffff); cardMat.needsUpdate = true; });
  for (const f of [front, back]) { f.updateCurve(0, ring); f.moveAlongCurve(i / CARD_IMAGES.length); }
  frontScene.add(front.object3D); backScene.add(back.object3D);
  flows.push(front, back);
});

const DEG = Math.PI / 180;
function placeCameras(x) {
  const p = phone.matches;
  for (const c of [backCam, frontCam]) {
    c.position.set(p ? 4 : 0, p ? -17 : -14, -70);
    c.rotation.set((p ? -200 : -192) * DEG, 0, (p ? -38 : -25) * DEG + (x - 0.5) * 0.2);
  }
}
function resize() {
  const w = hero.clientWidth, h = hero.clientHeight;
  for (const r of [backR, frontR]) r.setSize(w, h, false);
  // fov widens on narrower screens: 28 * (max(0, 1.8717949 - aspect) * 0.5 + 1)
  const fov = 28 * (Math.max(0, 1.8717948717948718 - w / h) * 0.5 + 1);
  for (const c of [backCam, frontCam]) { c.aspect = w / h; c.fov = fov; c.updateProjectionMatrix(); }
}
new ResizeObserver(resize).observe(hero);
resize();

let inView = true;
new IntersectionObserver(([e]) => { inView = e.isIntersecting; last = performance.now() - 16; }).observe(hero);

let last = performance.now() - 16;
function tick(now) {
  requestAnimationFrame(tick);
  if (!inView) return;
  const dt = Math.min(now - last, 60); last = now;
  // pointer smoothing: lerp 0.05 per 16 ms frame, snap under 1/2500
  const k = 0.05 * dt / 16;
  smoothX = Math.abs(targetX - smoothX) < 1 / 2500 ? targetX : smoothX + (targetX - smoothX) * k;
  updateLetters(smoothX);
  placeCameras(smoothX);
  if (!reduce) for (const f of flows) f.moveAlongCurve(dt * -5e-5);
  backR.render(backScene, backCam);
  frontR.render(frontScene, frontCam);
}
requestAnimationFrame(tick);
</script>
</body>
</html>
```
