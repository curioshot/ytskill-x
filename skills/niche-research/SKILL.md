---
name: niche-research
description: Find profitable YouTube niches. Scores candidate niches by demand (views), competition (channel concentration) and engagement using the YouTube Data API v3. Use when asked to research, compare or pick a YouTube niche, validate a channel idea, or estimate niche profitability.
compatibility: Works in any agent with shell/curl + jq access. Requires YOUTUBE_API_KEY.
---

# Niche Research

Find and score profitable YouTube niches.

## Inputs

- Seed niche ideas (1+), e.g. `"personal finance"`, `"ai tools"`, `"home automation"`
- Optional: region code (default `US`), max results per seed (default 25)

## Requirements

`YOUTUBE_API_KEY` env var (YouTube Data API v3, enabled in Google Cloud). If missing, stop and tell the user how to create one; do not fabricate numbers.

## Workflow

1. **Expand seeds.** For each seed, brainstorm 3–5 adjacent long-tail seeds (e.g. "personal finance" → "finance for nurses", "frugal living", "investing for beginners"). Confirm the list with the user if it's large.
2. **Fetch top videos per seed** (order by view count):

   ```bash
   curl -s -G https://www.googleapis.com/youtube/v3/search \
     --data-urlencode "q=$SEED" \
     --data-urlencode "part=snippet" --data-urlencode "type=video" \
     --data-urlencode "order=viewCount" --data-urlencode "maxResults=25" \
     --data-urlencode "key=$YOUTUBE_API_KEY"
   ```

   Collect the `videoId`s.
3. **Fetch statistics** (batch 50 ids per call):

   ```bash
   curl -s "https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=$IDS&key=$YOUTUBE_API_KEY"
   ```

4. **Compute per seed** (jq or python3):
   - `avg_views` = mean viewCount
   - `unique_channels` = distinct channelId count
   - `competition` = 1 − (unique_channels / sample_size) — 0 = diverse, 1 = dominated by few channels
   - `engagement_per_1k` = (likes + comments) / views × 1000
   - `score` = round(log10(avg_views) × 10 × (1 − 0.6 × competition) × (1 + min(engagement_per_1k, 50)/50), 2)
5. **Rank** seeds by score. Flag niches with score ≥ 25 as strong, 15–25 as moderate, < 15 as weak.
6. **Write report** to `./project/niches-<date>.md` + `.json`: ranked table, per-seed top 5 videos (title, views, channel), and a one-line verdict each.
7. **Recommend** the top 1–3 niches and why, noting monetization angle (ads, affiliate, sponsors, products) — not from API data, from your own knowledge, clearly labeled as such.

## Quality checks

- Numbers come from real API responses only — never invent them.
- If quota is exhausted (HTTP 403 + "quota"), report it and stop; do not estimate from memory.
- Sample size < 10 videos → mark confidence "low".
- Abort seeds that return 0 results.

## Output

Markdown report + raw JSON. Always end with a recommended niche and the next suggested skill (`video-ideas`).
