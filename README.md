# ytskill-x

Agent skills for the full YouTube pipeline, following the [Agent Skills](https://agentskills.io) `SKILL.md` format. Works with opencode, Claude Code, Codex, Antigravity and any agent that loads skills from a directory.

| Stage | Skill |
|---|---|
| Orchestrator | [`skills/ytskill-x`](skills/ytskill-x/SKILL.md) |
| Niche research | [`skills/niche-research`](skills/niche-research/SKILL.md) |
| Trend research | [`skills/trend-research`](skills/trend-research/SKILL.md) |
| Competitor analysis | [`skills/competitor-analysis`](skills/competitor-analysis/SKILL.md) |
| Video ideas | [`skills/video-ideas`](skills/video-ideas/SKILL.md) |
| Scriptwriting | [`skills/scriptwriting`](skills/scriptwriting/SKILL.md) |
| Voiceover | [`skills/voiceover`](skills/voiceover/SKILL.md) |
| Video creation | [`skills/video-creation`](skills/video-creation/SKILL.md) |
| Thumbnails | [`skills/thumbnail`](skills/thumbnail/SKILL.md) |
| YouTube SEO | [`skills/youtube-seo`](skills/youtube-seo/SKILL.md) |
| Publishing | [`skills/publishing`](skills/publishing/SKILL.md) |
| Analytics | [`skills/analytics`](skills/analytics/SKILL.md) |

## Install

Copy any skill folder into your agent's skills directory, e.g.:

```bash
# opencode
cp -r skills/* ~/.opencode/skills/
# Claude Code
cp -r skills/* ~/.claude/skills/
# Codex / Antigravity: see your agent's skills docs; the format is identical
```

## Credentials

Skills call the YouTube Data API v3 and optional LLM/media providers. Copy `.env.example` to `.env` (or export the vars) — never commit it. Each skill states exactly which keys it needs and degrades gracefully without them.

## Conventions

- Every skill is self-contained `SKILL.md` (+ optional `references/`).
- `$YOUTUBE_API_KEY` = YouTube Data API v3 key. OAuth2 is required only for `publishing`.
- **Videos are made entirely by code + TTS** — ffmpeg/Pillow render every frame, narration via edge-tts (free) or ElevenLabs, sound effects via ffmpeg-synthesized audio or licensed royalty-free SFX (Freesound/Pixabay Audio). No GUI editor anywhere.
- Each video lives in `./project/<slug>/` and **must contain**:

  ```
  project/xyz/
    xyz.mp4           # the video — named after the folder
    title.txt         # final YouTube title
    description.txt   # final YouTube description
    thumbnail.png     # master, rendered/drawn in code (Pillow/ImageMagick)
    thumbnail.jpg     # JPEG <2MB for the YouTube API
  ```

  `title.txt` + `description.txt` + `<slug>.mp4` are the canonical upload payload consumed by `publishing`.
- `publishing` is dry-run by default; real uploads require explicit confirmation.

## License

MIT
