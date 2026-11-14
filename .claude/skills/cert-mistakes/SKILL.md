---
name: cert-mistakes
description: Spaced review of previously missed CCAR-F questions, re-asked in reworded form until mastered. Use when the user says "cert-mistakes" or wants to review their Claude certification mistakes.
argument-hint: "[count, default all open (max 15)] [domain filter e.g. d2]"
---

# /cert-mistakes: fix what you got wrong

Source: `my-mistakes.md` (`## Open` / `## Mastered`). If it doesn't exist yet (or `## Open` is empty), say so and suggest `/cert-drill weak`.

## Run
1. Read `## Open`. Apply the optional count or domain filter from `$ARGUMENTS`. Order: oldest first, then lowest streak.
2. For each entry, **rewrite** the question before asking it:
   - Same concept, different scenario or wording.
   - Reorder the options so the correct letter moves.
   - Swap in at least one new distractor.
   - The goal is to test the rule, not whether the user remembers the letter.
3. Ask with AskUserQuestion, one at a time. After each answer:
   - ✅ Bump the streak (`0/2` → `1/2` → `2/2`). At 2/2, move the entry to `## Mastered` with today's date.
   - ❌ Reset to `0/2`. Re-explain the concept from a different angle than the stored rule (a new analogy, or a concrete config/code example), and update the entry's rule text if the user's confusion was different this time.
4. Append a row to `my-progress.md` with activity `mistakes review` and the score.
5. End with: how many mastered today, how many still open, and which concept recurs most among the open entries.
