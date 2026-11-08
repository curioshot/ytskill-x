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
- All generated artifacts go to `./project/<slug>/`.
- `publishing` is dry-run by default; real uploads require explicit confirmation.

## License

MIT
