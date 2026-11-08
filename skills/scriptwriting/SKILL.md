---
name: scriptwriting
description: Write retention-focused YouTube scripts. Produces hook → open loop → structured sections → payoff → CTA scripts with timing marks, optimized for average view duration. Use when asked to write a YouTube script, video script, or improve retention for a concept.
compatibility: Works in any agent; no external keys required.
---

# Scriptwriting

Write scripts people actually watch to the end.

## Inputs

- Chosen idea/title (required, from `video-ideas` or the user)
- Format: `longform` (default) or `shorts`
- Target length (longform default 8–12 min ≈ 1200–1800 words of VO)
- Audience, tone (default: conversational expert), language

## Workflow

1. **Load context.** Read `./project/<slug>/ideas.md` if it exists. Re-derive the angle and the promise the title makes — the script must deliver exactly that promise.
2. **Outline first** (show it to the user for longform):
   - `HOOK (0–15s)` — open with the strongest moment/result/question; never "hey guys, welcome back"
   - `OPEN LOOP (15–45s)` — state what the viewer will learn and tease the payoff ("by the end you'll know X — most people get step 2 wrong")
   - `BODY` — 3–7 sections, each: claim → evidence/example → mini payoff; escalate difficulty/interest
   - `MID-VIDEO REHOOK (~50%)` — re-open a loop before the natural drop-off point
   - `PAYOFF (final ~10%)` — deliver the promised result clearly
   - `CTA` — one action only (subscribe/next video/product), ≤ 15s
3. **Write the VO script** with timing marks every section: `[0:00]`, `[0:20]`… Keep sentences short (≤ 20 words). Add `[B-ROLL: ...]` and `[ON-SCREEN TEXT: ...]` cues where they carry meaning.
4. **Retention passes** (do all three, silently):
   - Cut throat-clearing first sentences; start mid-energy
   - Add a curiosity gap every 30–45s
   - Kill repetition and passive voice; read the hook aloud — if it's boring to say, rewrite it
5. **Shorts variant**: ≤ 60s, single idea, hook in first 2s, text-on-screen friendly, vertical framing cues.
6. **Write files**: `./project/<slug>/script.md` (full script with cues) and the outline kept at the top. Include word count and estimated duration (words ÷ 150 wpm).

## Quality checks

- Timing marks must sum to target length ±15%.
- No section without a payoff; no hook later than 15s in.
- Facts/claims that would be risky to state as fact must be hedged or sourced — never fabricate stats.
- CTA is singular and specific.
- If the idea is unanswerable from available information, say what research would be needed instead of inventing.

## Output

`./project/<slug>/script.md`. End with estimated duration and route to `voiceover`.
