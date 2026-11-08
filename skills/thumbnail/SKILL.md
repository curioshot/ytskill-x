---
name: thumbnail
description: Generate YouTube thumbnail concepts and render them. Produces CTR-optimized concepts (text ≤ 4 words, face/emotion, one focal point) and, if tooling allows, a rendered 1280×720 image. Use when asked for thumbnails, thumbnail ideas, or a clickable cover image.
compatibility: Agent with shell access. Pillow/ImageMagick or an image-generation tool for rendering; otherwise returns production-ready concept prompts.
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
4. **Render** (if `render` mode and tooling exists):
   - With Pillow: gradient/solid background → optional face cutout centered-right → bold text left in all-caps with thick outline/shadow → export `1280×720` JPEG quality 90 to `./project/<slug>/thumbnail.jpg`. Compress < 2MB (YT limit).
   - With an image generator: ready-to-paste prompt including all the above (style, colors, text, composition, 1280×720).
   - If neither exists: output the full production brief + prompt; do not pretend an image was made.
5. **Variants**: produce 2 variants for A/B testing (different text or face). YouTube supports A/B test thumbnails — note this in output.
6. **Verify** rendered file: exists, 1280×720, < 2MB; open it (Read tool if available) and check text isn't clipped and contrast is readable.

## Quality checks

- ≤ 4 words; if it needs more, the concept is wrong.
- No text smaller than ~20% of frame height; outline/stroke mandatory for legibility.
- One clear focal point — no collage clutter.
- Thumbnail must be truthful to the video's content (no fake faces of real people; AI faces must be obviously stylized or consented).

## Output

`./project/<slug>/thumbnail.jpg` (or concept brief) + the chosen concept described in `./project/<slug>/thumbnail.md`. Route to `youtube-seo`.
