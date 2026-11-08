---
name: competitor-analysis
description: Analyze successful YouTube channels. Pulls channel stats, recent uploads and outlier videos (>2x channel average) via the YouTube Data API v3. Use when asked to analyze a competitor channel, spy on a niche leader, find what works for a channel, or benchmark channel performance.
compatibility: Works in any agent with shell/curl + jq access. Requires YOUTUBE_API_KEY.
---

# Competitor Analysis

Tear down a YouTube channel: what it is, how fast it posts, and which videos beat its own average (proves the packaging/topic, not the channel size, drove performance).

## Inputs

- Channel handle(s) (`@name`), channel id, or channel URL
- Optional: number of recent videos to analyze (default 25)

## Requirements

`YOUTUBE_API_KEY` env var. If missing, stop and tell the user.

## Workflow

1. **Resolve channel** by handle:

   ```bash
   curl -s "https://www.googleapis.com/youtube/v3/channels?part=snippet,statistics,contentDetails&forHandle=NAME&key=$YOUTUBE_API_KEY"
   ```

   If that fails, search `channels?part=snippet&type=...&q=` fallback via `search&type=channel`.
2. **Record**: subscriberCount, viewCount, videoCount, title, publishedAt.
3. **Get uploads playlist id** from `contentDetails.relatedPlaylists.uploads`, then list recent uploads:

   ```bash
   curl -s "https://www.googleapis.com/youtube/v3/playlistItems?part=contentDetails&playlistId=UPLOADS_ID&maxResults=25&key=$YOUTUBE_API_KEY"
   ```
4. **Batch fetch video stats** for those ids (`videos?part=snippet,statistics`).
5. **Compute**:
   - `recent_avg_views` (mean), `recent_median_views`
   - `upload_cadence` = videos per week over the analyzed window
   - `outlier_videos` = recent videos with views > 2 × recent_avg_views
6. **Deconstruct outliers** (this is the valuable part): for each outlier, note the *topic pattern* (searchable subject vs novel angle), *title formula*, *duration*, *publish recency*. You cannot see thumbnails via API — ask the user to eyeball them or note thumbnails manually.
7. **Recurring hits**: which topics/formulas repeat across the channel's best performers.
8. **Write report** to `./project/competitor-<handle>.md` + `.json`: channel summary, cadence, outliers table, repeatable playbook (3–7 bullet formulas), and gaps the channel is not covering.

## Quality checks

- All figures from API responses; channel-not-found must produce a clear error, never a guessed channel.
- Outliers defined against the channel's *own* baseline, not absolute views.
- Handle private/zero-stat channels (subscriberCount hidden → report "hidden").
- Upload cadence over mixed-age windows is misleading — state the window analyzed.

## Output

Competitor report + JSON. End with: "Steal these N formulas" list and route to `video-ideas`.
