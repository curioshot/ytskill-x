---
name: ytskill-x
description: Orchestrate the full YouTube pipeline end to end — niche research, trends, competitors, ideas, scripts, voiceover, video, thumbnails, SEO, publishing, analytics. Use when asked to run/start/build a YouTube video or channel workflow, "do the whole pipeline", or when unsure which single stage to use. Delegates to the stage skills: niche-research, trend-research, competitor-analysis, video-ideas, scriptwriting, voiceover, video-creation, thumbnail, youtube-seo, publishing, analytics.
compatibility: Agent with shell access. Designed to load alongside the per-stage ytskill-x skills.
---

# ytskill-x — YouTube Pipeline Orchestrator

Run the whole machine, step by step, with the user in the loop at the gates that matter.

## When to use

- "Make me a faceless YouTube video about X" / "build my channel" / "run the pipeline" → this skill.
- Narrow requests ("just trending topics") → go straight to the single stage skill instead.

## Global conventions

- Artifacts live in `./project/<slug>/` (`<slug>` = slugified topic). Create it on first artifact.
- `./project/<slug>/learnings.md` accumulates channel learnings; **read it at the start of every run** and apply it.
- Keys: `YOUTUBE_API_KEY` (research/analytics), OAuth tokens (publishing/analytics private metrics), optional `ELEVENLABS_API_KEY`, `PEXELS_API_KEY`. LLM creativity = the host agent itself; no LLM key required.
- Every stage ends by naming the next stage. Never skip silently.

## Pipeline

| # | Stage | Skill | Needs user gate? |
|---|---|---|---|
| 1 | Niche research | niche-research | Yes — user picks the niche |
| 2 | Trend research | trend-research | No (report feeds step 3) |
| 3 | Competitor analysis | competitor-analysis | No |
| 4 | Video ideas | video-ideas | Yes — user picks the idea |
| 5 | Scriptwriting | scriptwriting | Yes — approve outline before VO |
| 6 | Voiceover | voiceover | No |
| 7 | Video creation | video-creation | Optional asset approval |
| 8 | Thumbnail | thumbnail | Yes — user picks concept |
| 9 | YouTube SEO | youtube-seo | No |
| 10 | Publishing | publishing | **Mandatory confirmation** — default private |
| 11 | Analytics | analytics | No (post-publish review) |

## Workflow

1. **Brief.** Get: niche/channel goal, format (longform/shorts), faceless or on-camera, cadence, languages. Create `learnings.md` if absent; read it if present.
2. **Stages 1–3 (research).** Run in order; each writes `./project/<slug>/<stage>.md` and `.json`. Present the combined research summary — max 20 lines — and **stop**: user confirms niche.
3. **Stage 4 (ideas).** Generate 10 scored ideas; **stop**: user picks one.
4. **Stage 5 (script).** Outline first → user approves → full script. Re-show the hook; rewrite until user is happy.
5. **Stages 6–7 (production).** Voiceover → video. Report duration/size. If Pexels stock is unavailable, fall back to slideshow and tell the user.
6. **Stage 8 (thumbnail).** 3–5 concepts, user picks; render if possible.
7. **Stage 9 (SEO).** Recommended title + description + chapters + tags.
8. **Stage 10 (publish).** Always dry-run first. Upload only after explicit user confirmation; default privacy `private`. Record `publish.json` (videoId).
9. **Stage 11 (analytics).** Schedule/run after 7 days: pull metrics, diagnose, append to `learnings.md`, propose the next video from what won.

## Partial runs

User may enter mid-pipeline. If `./project/<slug>/` already has later-stage artifacts, resume from the earliest missing/stale stage instead of restarting.

## Honesty rules

- Never invent API numbers, search volumes, retention or CTR — measured data only; gaps are reported as "unavailable".
- AI-generated content must be reviewed by the user before publishing; never auto-publish.
- Respect YouTube policies: made-for-kids accuracy, no cloned voices without rights, licensed music/stock only.
