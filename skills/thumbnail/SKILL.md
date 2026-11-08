---
name: thumbnail
description: Generate YouTube thumbnail concepts and render them. Produces CTR-optimized concepts (text ≤ 4 words, face/emotion, one focal point) and, if tooling allows, a rendered 1280×720 image. Use when asked for thumbnails, thumbnail ideas, or a clickable cover image.
compatibility: Agent with shell access. Prefers coded rendering (Python + Pillow or ImageMagick) — required when the model is NOT vision capable; image-generation tools optional. Produces BOTH thumbnail.png and thumbnail.jpg.
---

# Thumbnail

Concept and render clickable thumbnails.

## Inputs

- Video title/angle (required — from `ideas.md`/`seo` output)
- Optional: brand colors, logo file, channel face photo (`./project/<slug>/assets/`)
- Optional: output mode `concept` (default) or `render`

## Workflow

1. **Extract the promise.** One line: what will the viewer get? The thumbnail must amplify that, not restate the title.
2. **Generate 3–5 concepts.** Each concept =
   - `text` — ≤ 4 words, high contrast, readable at mobile size (test at ~140px wide)
   - `visual` — exactly one focal point: face w/ emotion | big object | before/after split | number/stat
   - `composition` — left/right thirds, rule-of-thirds placement, empty zone for YT duration stamp
   - `palette` — 2–3 colors max; avoid YouTube-red backgrounds (blends into UI)
   - `ctr_rationale` — why a scrolling human would tap (curiosity gap, face reaction, number, contrast)
   - `risk` — any deception/accuracy issue (flag; don't produce misleading thumbnails)
3. **Score and pick** the best concept with the user (or auto-pick if user said "just decide").
4. **Render in code** (always — the model is often not vision capable, so the thumbnail must be drawable without "seeing"):
   - With Pillow (default): gradient/solid background → optional face cutout centered-right (only if a photo exists in `assets/`) → bold text left in all-caps with thick outline/shadow → export **both**: `./project/<slug>/thumbnail.png` (master) and `./project/<slug>/thumbnail.jpg` (JPEG quality 90, < 2MB, YT limit). 1280×720.
   - With ImageMagick: `convert` equivalent (`-size 1280x720 gradient:... -annotate ...`), same two outputs.
   - Only if a vision/image-capable model AND image tool exist: an AI image for extra polish — then still export both png+jpg and keep the coded version as fallback.
   - If no image tooling at all: output the full production brief; do not pretend an image was made.
5. **Variants**: produce 2 variants for A/B testing (different text or face). YouTube supports A/B test thumbnails — note this in output.
6. **Verify** both files: exist, 1280×720, jpg < 2MB; if the agent can view images, open thumbnail.jpg and check text isn't clipped and contrast is readable. If not vision capable, verify geometrically — text bbox inside canvas, contrast ratio computed in code (e.g. WCAG contrast ≥ 4.5:1 for the text vs background).

## Quality checks

- ≤ 4 words; if it needs more, the concept is wrong.
- No text smaller than ~20% of frame height; outline/stroke mandatory for legibility.
- One clear focal point — no collage clutter.
- Thumbnail must be truthful to the video's content (no fake faces of real people; AI faces must be obviously stylized or consented).

## Output

`./project/<slug>/thumbnail.png` **and** `./project/<slug>/thumbnail.jpg` (both mandatory — jpg for the YouTube API, png as master) + the chosen concept described in `./project/<slug>/thumbnail.md`. Route to `youtube-seo`.
