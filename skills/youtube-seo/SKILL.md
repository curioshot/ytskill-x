---
name: youtube-seo
description: Optimize YouTube titles, tags, descriptions and chapters. Generates keyword-backed metadata from the script for search + browse + suggested traffic. Use when asked for YouTube SEO, titles, tags, description, chapters, or to optimize a video's metadata.
compatibility: Agent with shell access. YOUTUBE_API_KEY optional (search volumes are not public; API is used to sanity-check demand, not to invent volumes).
---

# YouTube SEO

Package the video for discovery.

## Inputs

- `./project/<slug>/script.md` (required) and optionally `ideas.md`, thumbnail concept
- Optional: target primary keyword, language

## Workflow

1. **Extract candidates** from the script: exact phrases viewers would search (how-to, problem, product, comparison). List 8–12 candidates.
2. **Demand sanity check** (if `YOUTUBE_API_KEY` set): for each candidate, run a `search?q=...&type=video&order=viewCount&publishedAfter=<12mo>` — gauges content exists + top hits' views. If a candidate has zero/recent-low competition with demand, it's a long-tail win. Never state fake search volumes ("10K searches/month") — YouTube doesn't expose them; say "high view ceiling observed on top results, low competition" style phrasing.
   - Script: `for q in ...; do curl -s -G ... --data-urlencode "q=$q" ...; done`
3. **Pick** 1 primary keyword (best demand × relevance × attainable) + 2–4 secondary.
4. **Titles** — produce 5 variants:
   - ≤ 60 chars; primary keyword in first half; one promise; no `|` spam; no ALL-CAPS except 1 word max; avoid deception
   - include one `[year]`/authority signal variant if it fits
5. **Description** template:
   - Line 1–2: primary keyword naturally + what the video delivers (this shows in search)
   - Line 3: CTA subscription line
   - Body: 3–5 short paragraphs with secondary keywords (no keyword stuffing)
   - Chapters list (from `[0:00]` marks in script.md, format `0:00 Section`)
   - Links/resources/social placeholders marked `TODO(user)`
   - 3–5 hashtags (first 3 render above title; `#shorts` only for shorts)
6. **Tags** — 8–15: exact match primary, long-tails, channel-relevant. Tags matter least; say so.
7. **Write** `./project/<slug>/seo.json`: `{ titles: [...], description, tags: [...], primary_keyword, secondary_keywords: [...], chapters: [...] }` + human-readable `seo.md`.

## Quality checks

- Primary keyword appears naturally in line 1 of description and in the recommended title — no stuffing.
- Chapters match the script's timing marks (first chapter at 0:00, ≥ 3 chapters, ascending times).
- No claims unsupported by the video; no fake links; user-fill TODOs are explicit.
- Description ≤ 5000 chars.

## Output

`seo.md` + `seo.json`. End with recommended title + route to `publishing`.
