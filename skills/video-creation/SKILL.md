---
name: video-creation
description: Automate YouTube video production entirely with code and TTS — no GUI editor. Renders the finished MP4 with ffmpeg from voiceover, stock/coded visuals, captions and sound effects (Freesound/Pixabay Audio or ffmpeg-synthesized SFX). Use when asked to create, render, produce or assemble a YouTube video, or turn a script+voiceover into a video file.
compatibility: Agent with shell access; requires ffmpeg and a voiceover (see voiceover skill). PEXELS_API_KEY optional for stock footage; FREESOUND_API_KEY optional for SFX.
---

# Video Creation

Assemble a finished video from voiceover + visuals + captions — **entirely by code**: every frame is rendered programmatically (ffmpeg / Pillow), all narration comes from a TTS engine, and all sound effects from code-synthesized audio or a royalty-free SFX library. No manual video editing anywhere in the pipeline.

## Tools used (all code, all scriptable)

| Need | Tool |
|---|---|
| Narration (TTS) | `edge-tts` (free, pip) or ElevenLabs API (`ELEVENLABS_API_KEY`) — see `voiceover` skill |
| Render / concat / pad / mix | `ffmpeg` + `ffprobe` |
| Slideshow cards & motion graphics | Python 3 + Pillow |
| Captions (.srt) | edge-tts `--write-subtitles`, or `openai-whisper` if missing |
| Stock footage/images | Pexels API (`PEXELS_API_KEY`) via curl |
| Sound effects | Freesound API (`FREESOUND_API_KEY`) or Pixabay Audio via curl; or ffmpeg-synthesized SFX (`aevalsrc`/`anoisesrc`) when no key/asset |
| Audio mixing (music/SFX under VO) | `ffmpeg` `amix`/`sidechaincompress` or `pydub` |

## Inputs

- Project dir `./project/<slug>/` (the folder is named after the video, e.g. `xyz`) with `script.md` and `voiceover.mp3` (+ `.srt`)
- This skill writes `./project/<slug>/<slug>.mp4` — the final video is named after the folder
- Visual strategy: `stock` (Pexels), `ai` (if a vision/image-capable model is available), `local` (assets folder), or `slideshow` (coded title cards/text screens — the default when the model is **not** vision capable) — ask the user if unspecified
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
   - `ai`: only if a vision/image-capable model + image generator exists; otherwise use `slideshow`.
   - `local`: use `./project/<slug>/assets/`; if empty, ask the user.
   - `slideshow`: render every frame in code with Pillow — gradients, big type, shapes, key phrases from the script. No model vision required.
4. **Sound effects** (from `[SFX: ...]` cues in the script): for each cue, fetch a matching royalty-free SFX (`freesound.org` with `FREESOUND_API_KEY`, or Pixabay Audio) into `./project/<slug>/assets/sfx/` — or synthesize with ffmpeg, e.g. a whoosh: `ffmpeg -f lavfi -i "anoisesrc=d=0.4:c=pink:a=0.3,highpass=f=400" sfx-whoosh.wav`. If neither source works, report which cues are uncovered; never substitute copyrighted audio.
5. **Captions**: use the `.srt` from the voiceover skill (or generate from the mp3 via Whisper if missing: `pip install openai-whisper && whisper voiceover.mp3 --output_format srt`). Burn in only if the style suits; otherwise upload as a separate track later.
6. **Render** (16:9 example — scale/pad each asset to 1920×1080, concat, overlay VO):

   ```bash
   ffmpeg -f concat -safe 0 -i list.txt -i voiceover.mp3 \
     -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
     -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest <slug>.mp4
   ```

   `slideshow` fallback: `python3 -m pip install pillow` → render PNG cards (1080p, big type, key phrase per section) → ffmpeg same pipeline.
   Shorts: same commands with `1080x1920` and `setsar=1`.
7. **Caption burn-in** (optional, great for shorts): `-vf "subtitles=voiceover.srt:force_style='FontSize=18,PrimaryColour=&H00FFFFFF'"`.
8. **Mix SFX + music under VO**: for each SFX at its cue timestamp, `ffmpeg -i <slug>.mp4 -i assets/sfx/whoosh.wav -filter_complex "[1:a]adelay=2000|2000[s];[0:a][s]amix=inputs=2:duration=first:weights=1 0.25[a]" -map 0:v -map "[a]" <slug>.with-sfx.mp4`, then `mv <slug>.with-sfx.mp4 <slug>.mp4`. Music at ~−25 LUFS with VO dominant; prefer `sidechaincompress` for ducking.
9. **Verify output**: `ffprobe` <slug>.mp4 — duration ≈ VO duration ±2s, streams: h264 video + aac audio, resolution correct. Report file size.

## Quality checks

- No unlicensed music/images; stock must come from Pexels/Pixabay/public domain — no YouTube rips.
- Video duration must not exceed VO duration (no dead air; `-shortest` enforced).
- Check for missing assets before render — a black 10s gap is a defect.
- If ffmpeg fails, paste the *last 15 lines* of the error and fix the filter/paths; don't blindly retry.

## Output

`./project/<slug>/<slug>.mp4` — the video is named after the folder. Report resolution, duration, size, caption status, which SFX cues were covered. Route to `thumbnail` and `youtube-seo`.
