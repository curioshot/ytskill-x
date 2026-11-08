---
name: voiceover
description: Generate AI narration for a YouTube script. Produces a voiceover audio file from script.md using edge-tts (free, no key) or ElevenLabs (premium). Use when asked to generate narration, voice over, TTS, or an audio track for a video.
compatibility: Agent with shell access. edge-tts via pip (free) or ELEVENLABS_API_KEY for premium voices.
---

# Voiceover

Turn the script into narration audio.

## Inputs

- Path to script (default `./project/<slug>/script.md`)
- Voice choice: default `en-US-GuyNeural` (edge-tts) — pick to match channel tone; premium `elevenlabs` if `ELEVENLABS_API_KEY` is set and user wants it
- Optional: pace (default +0%), output name (default `voiceover`)

## Workflow

1. **Extract VO text.** Strip timing marks, `[B-ROLL: ...]`, `[ON-SCREEN TEXT: ...]` and headings from the script — keep only spoken lines. Save as `./project/<slug>/voiceover.txt`. Verify it reads naturally aloud; fix awkward breath-less run-ons.
2. **Check tooling**: `python -c "import edge_tts"` (else `pip install edge-tts`). For ElevenLabs, confirm `ELEVENLABS_API_KEY` is set.
3. **Generate**:

   ```bash
   # edge-tts (free) — also emits Word-level subtitles
   edge-tts --file ./project/<slug>/voiceover.txt \
     --voice en-US-GuyNeural \
     --write-media ./project/<slug>/voiceover.mp3 \
     --write-subtitles ./project/<slug>/voiceover.srt
   ```

   ```bash
   # ElevenLabs (premium)
   curl -s -X POST "https://api.elevenlabs.io/v1/text-to-speech/$VOICE_ID" \
     -H "xi-api-key: $ELEVENLABS_API_KEY" -H "Content-Type: application/json" \
     -d '{"text": "...", "model_id": "eleven_multilingual_v2"}' --output ./project/<slug>/voiceover.mp3
   ```

4. **Split by section** if the video is long (optional): keep one master `voiceover.mp3` and one `.srt` — downstream `video-creation` uses both.
5. **Verify**: file exists, duration ≈ script word count ÷ 150 wpm ±20%. Report the duration. Re-generate with different voice/rate once if the user dislikes it.
6. **Audio hygiene**: loudness ≈ −14 LUFS for YouTube: `ffmpeg -i voiceover.mp3 -af loudnorm=I=-14:TP=-1.5:LRA=11 voiceover-norm.mp3`.

## Quality checks

- Never ship the `.srt`/cues text inside the VO.
- MP3 must play to the end — corrupt/truncated files get regenerated, not patched.
- Pronunciation of brand names: add `--rate` or phonetic hints in `voiceover.txt` (e.g. "GitHub" → "git hub") rather than re-recording.
- Do not pick a cloned real person's voice without rights.

## Output

`voiceover.mp3` (+ `.srt` for captions) in `./project/<slug>/`. Report duration and voice used. Route to `video-creation`.
