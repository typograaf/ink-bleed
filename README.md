# INK — Bleed / Choke

A type tool for a localized ink-bleed effect. Type stays crisp, and inside a
draggable radial falloff the letters bleed (spread/merge into ink) or choke
(erode). Export the result as a standalone SVG.

## Use
Open `index.html` directly in a browser (double-click) — no server needed.

- **Text** — what to set. Press **Enter** for a hard return / multiple lines
  (centered and stacked).
- **Font** — drop any `.ttf`/`.otf`/`.woff` **anywhere onto the window**, or use the
  **Upload font** button. Default is Space Grotesk.
- **Font size** — type size.
- **Leading** — line spacing (% of the natural line height), for multi-line text.
- **Tracking** — letter spacing (kerning-aware; negative tightens).
- **Drag anywhere on the canvas** — move the falloff (the dashed ring). Its centre
  is where the bleed is strongest.
- **Scroll / mouse-wheel over the canvas** — resize the falloff.
- **Falloff size** — radius of the affected region (or just scroll).
- **Falloff softness** — how much of the core is full-strength before fading out.
- **Falloff bleed (− choke / + bleed)** — the localized effect, scaled by the
  falloff: `+` spreads/merges the ink outward, `−` chokes (erodes) it. Strongest
  at the centre, fading to nothing at the ring.
- **Overall thickness (− / +)** — a global stroke weight applied everywhere,
  independent of the falloff. It's rounded by Roundness too (not a hard stroke).
- **Roundness in falloff** / **Roundness outside falloff** — two independent
  amounts of corner-rounding + overlap-fusing (organic ink shapes), blended by the
  falloff. Set outside to 0 for crisp type with rounded ink only under the
  falloff, or both equal for uniform rounding. Neither changes thickness.
- **Invert** — black-on-white ↔ white-on-black.
- **File name** — name the downloaded file here.
- **Export SVG** — downloads a standalone `.svg`: a single solid-black vector
  silhouette (real outline paths, no filters, no transparency), exactly what you
  see on screen. Opens cleanly in Illustrator and any browser.
- **Save settings / Load settings** — saves all controls (text, sliders, falloff
  position, invert) to a `<filename>.json` you can reload later to restore a look.
  The font itself isn't stored — keep using the current font or re-load it.

## How it works
The result is always one flat 100% black silhouette — never a blurred or
semi-transparent layer.

- `opentype.js` rasterizes the font + text to a supersampled coverage map.
- A **signed distance field** (exact Felzenszwalb–Huttenlocher EDT) is built from
  it. Dilating the silhouette is then just "fill where `distance < offset`" —
  true ink spread that rounds corners and merges neighbours without ever erasing
  thin strokes (a blur+threshold would thin them away).
- The offset varies in space: `offset = thickness + bleed · falloffWeight`, so the
  falloff drives how much each region bleeds (or chokes).
- **Roundness** blurs the distance field (a **smooth-union**): corners round and
  nearby/overlapping strokes fuse with organic ink necks into new shapes, and it's
  thickness-neutral (a field blur leaves straight edges in place). The field is
  blurred by the inside and outside amounts separately and blended by the falloff,
  so rounding can differ inside vs outside the ring.
- **Marching squares** traces the thresholded field back into smooth closed vector
  contours → one `<path>` (even-odd fill keeps the counters as holes).
- The live preview *is* that path, so export is just wrapping it in an `<svg>`.

## Files
- `index.html` — the whole app (SDF + marching-squares engine inline).
- `opentype.min.js` — font outline parser (vendored).
- `default-font.js` — Space Grotesk, base64-inlined so it loads with no network.
- `SpaceGrotesk.ttf` — source of the inlined default (kept for regeneration).

## Note on resolution
The whole word is traced as one silhouette, so even unaffected letters are
re-tessellated from the raster (corners are within ~1px of the original). Bump
`SS` / lower `STEP` in `index.html` for finer output at some CPU cost.
