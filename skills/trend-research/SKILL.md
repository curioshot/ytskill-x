---
name: trend-research
description: Discover trending topics on YouTube. Pulls mostPopular charts and recent high-velocity search results via the YouTube Data API v3. Use when asked what's trending, which topics are hot right now, what to make a video about, or to find rising/viral content in a niche.
compatibility: Works in any agent with shell/curl + jq access. Requires YOUTUBE_API_KEY.
---

# Trend Research

Surface trending topics worth making videos about — now, not last month.

## Inputs

- Region code (default `US`)
- Optional: category id (e.g. `27` = Education, `28` = Science & Tech, `22` = People & Blogs)
- Optional: niche keyword to scope the trend hunt (e.g. `"ai agents"`)
- Optional: lookback window (default 14 days)

## Requirements

`YOUTUBE_API_KEY` env var. If missing, stop and tell the user how to create one.

## Workflow

1. **Chart pull** — most popular right now:

   ```bash
   curl -s "https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&chart=mostPopular&regionCode=US&maxResults=25&key=$YOUTUBE_API_KEY"
   ```

   Add `&videoCategoryId=<id>` to scope by category.
2. **Velocity scan** (if a niche keyword is given) — recent uploads sorted by date, then re-ranked by views:

   ```bash
   curl -s -G https://www.googleapis.com/youtube/v3/search \
     --data-urlencode "q=$KEYWORD" --data-urlencode "part=snippet" \
     --data-urlencode "type=video" --data-urlencode "order=date" \
     --data-urlencode "publishedAfter=$(date -u -d '14 days ago' +%Y-%m-%dT%H:%M:%SZ)" \
     --data-urlencode "regionCode=US" --data-urlencode "maxResults=50" \
     --data-urlencode "key=$YOUTUBE_API_KEY"
   ```

   Then batch-fetch `videos?part=statistics` for the ids and sort by views.
3. **Velocity score** per video: `views / max(hours_since_publish, 1)`. High views with low hours = breakout topic.
4. **Cluster** the results into 3–7 topic clusters (manual/LLM grouping of titles). Drop hardware launches/news that won't have repeatable content potential unless the user's niche is news.
5. **Classify** each cluster: `evergreen-rising` (recurring demand), `viral-spike` (one-off), `seasonal`. Prefer evergreen-rising for channel building.
6. **Write report** to `./project/trends-<date>.md` + `.json`: clusters, velocity scores, example videos, and which clusters match the user's niche (if provided).

## Quality checks

- Use real API data only. On HTTP 403 quota errors, report and stop.
- Note that `mostPopular` is view-count based, not truly real-time; say so in the report.
- Distinguish "trending" (chart) from "breaking out" (velocity) — they are not the same list.
- Dates/hours must come from the API's `publishedAt`, never guessed.

## Output

Ranked clusters with evidence (titles, views, velocity). End with 3 recommended topics and route to `video-ideas`.
