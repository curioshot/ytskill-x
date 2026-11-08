---
name: video-ideas
description: Generate viral YouTube video concepts. Turns niche research, trend clusters and competitor outliers into scored, ready-to-produce video ideas with title, hook and angle. Use when asked for video ideas, what to make next, viral concepts, or to brainstorm a content calendar.
compatibility: Works in any agent; no external keys required. Best after niche-research, trend-research or competitor-analysis.
---

# Video Ideas

Generate viral-ready video concepts from evidence, not vibes.

## Inputs

- Topic/niche (required)
- Optional: paths to prior reports in `./project/` (niches, trends, competitor analysis) — read them if present
- Optional: content format (`longform` | `shorts`), target audience, number of ideas (default 10)
- Optional: channel constraints (faceless, budget, language)

## Workflow

1. **Load evidence.** Read any `./project/niches*.json`, `trends*.json`, `competitor-*.json`. Extract: proven outliers (>2x avg views), rising clusters, strong-scoring niches. If none exist, ask the user for 1–2 reference channels or run `competitor-analysis` first — ideas grounded in data outperform guesses.
2. **Extract formulas** from the evidence: title patterns (curiosity gap, number list, comparison, "X vs Y", contrarian take), outlier topic categories, audience intents (learn / solve / be entertained).
3. **Generate** the requested number of ideas. For each:
   - `title` — ≤ 60 chars, one clear promise, primary keyword near the front
   - `hook` — the first spoken line (must work in the first 5 seconds)
   - `angle` — what makes this different from the top 5 existing results
   - `format` — longform or shorts, estimated duration, style (listicle, tutorial, story, etc.)
   - `evidence` — which outlier/cluster/niche datum inspired it
   - `score` /25 — sum of: demand signal (0–10), competition gap (0–5), packaging strength (0–5), production ease (0–5)
4. **Deduplicate** near-identical angles; kill weak ideas (score < 15).
5. **Rank** by score and label the top 3 as ⭐ picks with one-line rationale.
6. **Write** to `./project/<slug>/ideas.md` where `<slug>` = slugified chosen topic: ranked numbered list + a table (title | format | score | evidence).

## Quality checks

- Every idea cites its evidence source; an idea with no data anchor must be labeled `[speculative]`.
- No idea copied verbatim from another channel's title — angles must be re-framed, not cloned.
- Titles must not be deceptive/clickbait that the video can't deliver — flag anything that is.
- Mix evergreen and timely ideas (at least 30% evergreen).

## Output

`./project/<slug>/ideas.md`. End with the ⭐ top-3 and route to `scriptwriting`.
