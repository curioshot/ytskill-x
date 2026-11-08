---
name: video-creation
description: Automate YouTube video production. Combines voiceover, stock/AI or local assets and captions into a finished MP4 with ffmpeg. Use when asked to create, render, produce or assemble a YouTube video, or turn a script+voiceover into a video file.
compatibility: Agent with shell access; requires ffmpeg and a voiceover (see voiceover skill). PEXELS_API_KEY optional for stock footage.
---

# Video Creation

Assemble a finished video from voiceover + visuals + captions.

## Inputs

- Project dir `./project/<slug>/` with `script.md` and `voiceover.mp3` (+ `.srt`)
- Visual strategy: `stock` (Pexels), `ai` (image generation via available tool), `local` (assets folder), or `slideshow` (title cards/text screens) — ask the user if unspecified; default `stock` with `slideshow` fallback
- Aspect: `16:9` longform (1920×1080) or `9:16` shorts (1080×1920)
- Optional: background music file (royalty-free, licensed — never rip tracks)

## Workflow

1. **Verify** `ffmpeg -version` and that `voiceover.mp3` exists and plays fully (`ffprobe` duration ≈ reported VO duration).
2. **Plan visuals per section** from the script's `[B-ROLL: ...]` cues: one visual per 3–8s. Write `./project/<slug>/visual-plan.md` (timestamp | cue | asset query/source).
3. **Fetch assets**:
   - `stock`: for each cue, query Pexels:

     ```bash
     curl -s -H "Authorization: $PEXELS_API_KEY" \
       "https://api.pexels.com/v1/videos/search?query=$QUERY&per_page=3"
     ```

     Download best match (landscape/Portrait per aspect) to `./project/<slug>/assets/`. If no key or no results, fall back to `slideshow` and tell the user.
   - `ai`: generate one 16:9/9:16 image per major section with the available image tool; save to assets.
   - `local`: use `./project/<slug>/assets/`; if empty, ask the user.
4. **Captions**: use the `.srt` from the voiceover skill (or generate from the mp3 via Whisper if missing: `pip install openai-whisper && whisper voiceover.mp3 --output_format srt`). Burn in only if the style suits; otherwise upload as a separate track later.
5. **Render** (16:9 example — scale/pad each asset to 1920×1080, concat, overlay VO):

   ```bash
   ffmpeg -f concat -safe 0 -i list.txt -i voiceover.mp3 \
     -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
     -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest final.mp4
   ```

   `slideshow` fallback: `python3 -m pip install pillow` → render PNG cards (1080p, big type, key phrase per section) → ffmpeg same pipeline.
   Shorts: same commands with `1080x1920` and `setsar=1`.
6. **Caption burn-in** (optional, great for shorts): `-vf "subtitles=voiceover.srt:force_style='FontSize=18,PrimaryColour=&H00FFFFFF'"`.
7. **Loudness**: already normalized in voiceover skill; if adding music, mix music at ~−25 LUFS under VO with `amix` and keep VO audible.
8. **Verify output**: `ffprobe` final.mp4 — duration ≈ VO duration ±2s, streams: h264 video + aac audio, resolution correct. Report file size.

## Quality checks

- No unlicensed music/images; stock must come from Pexels/Pixabay/public domain — no YouTube rips.
- Video duration must not exceed VO duration (no dead air; `-shortest` enforced).
- Check for missing assets before render — a black 10s gap is a defect.
- If ffmpeg fails, paste the *last 15 lines* of the error and fix the filter/paths; don't blindly retry.

## Output

`./project/<slug>/final.mp4`. Report resolution, duration, size, caption status. Route to `thumbnail` and `youtube-seo`.
