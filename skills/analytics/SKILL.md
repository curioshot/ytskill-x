---
name: analytics
description: Analyze YouTube video performance. Pulls views, watch time, average view percentage, CTR, likes, comments and subscriber impact via the YouTube Analytics API (OAuth) and Data API. Use when asked how a video performed, analyze channel stats, or review performance.
compatibility: Agent with shell/curl access. Data API needs YOUTUBE_API_KEY; deeper metrics (CTR, avg view %) need OAuth with scope yt-analytics.readonly.
---

# Analytics

Measure what actually happened, then feed it back into the next idea.

## Inputs

- Video id(s) or `./project/<slug>/publish.json` (from publishing skill); or "channel" for a full-channel overview
- Date range: `startDate`, `endDate` (default: video publish date → today; channel mode default: last 28 days)

## Requirements

- `YOUTUBE_API_KEY` — public stats (views/likes/comments).
- `GOOGLE_REFRESH_TOKEN` with scope `https://www.googleapis.com/auth/yt-analytics.readonly` — private metrics (estimated minutes watched, averageViewDuration, averageViewPercentage, impressions, impressionClickThroughRate, subscribersGained). If missing, provide public-metrics-only analysis and say exactly what's missing.

## Workflow

1. **Public stats** (no OAuth):

   ```bash
   curl -s "https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=$VID&key=$YOUTUBE_API_KEY"
   ```

2. **Access token** (OAuth mode): same refresh flow as the publishing skill.
3. **Analytics API v2 query** (per video):

   ```bash
   curl -s -G "https://youtubeanalytics.googleapis.com/v2/reports" \
     -H "Authorization: Bearer $ACCESS_TOKEN" \
     --data-urlencode "ids=channel==MINE" \
     --data-urlencode "metrics=views,estimatedMinutesWatched,averageViewDuration,averageViewPercentage,likes,comments,subscribersGained,impressions,impressionClickThroughRate" \
     --data-urlencode "dimensions=video" \
     --data-urlencode "filters=video==$VID" \
     --data-urlencode "startDate=$START" --data-urlencode "endDate=$END"
   ```

   Channel mode: drop `dimensions=video`, use last-28-days, add `sort=-views`.
4. **Compute derived signals**: hook strength = avg view % in first 30s if retention report available (`metrics=audienceWatchRatio&dimensions=elapsedVideoTimeRatio`); engagement rate = (likes+comments)/views × 1000; CTR health: < 2% packaging problem, 2–5% average, > 5% strong.
5. **Diagnose** (be decisive, use evidence):
   - Low CTR + low impressions → packaging/topic problem → route to `thumbnail` + `youtube-seo` (re-package: new title/thumbnail test)
   - Good CTR + low avg view % → intro/pacing problem → route to `scriptwriting` (tighten hook)
   - High view % + low impressions → distribution problem → route to `video-ideas` (broader packaging, Shorts repurpose)
6. **Write report** to `./project/<slug>/analytics-<date>.md` + `.json`: headline numbers, per-metric vs channel baseline, diagnosis, and 3 prioritized next actions with the skill to run next.
7. **Feed the loop**: append a one-line learning note to `./project/<slug>/learnings.md` (e.g. "contrarian titles outperform how-to in this niche — lean into contrarian packaging"). Future `video-ideas` runs must read it.

## Quality checks

- Never fabricate retention/CTR numbers — if a metric is unavailable, say "not available (needs yt-analytics.readonly scope)".
- Public view counts are point-in-time; note fetch time.
- small-sample warning when views < 100 — no conclusions, mark low confidence.
- Diagnose only from measured data; explicitly separate observation vs hypothesis.
