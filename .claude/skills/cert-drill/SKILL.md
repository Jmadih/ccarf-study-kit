---
name: cert-drill
description: Extra CCAR-F practice quiz on a chosen domain, scenario, or your weak spots, optionally timed like the real exam. Use when the user says "cert-drill" or wants more practice questions on a specific Claude certification topic.
argument-hint: "[d1-d5 | scenario: support, codegen, research, devprod, ci, extraction, conversational | weak | mixed] [count, default from my-plan.md, else 8] [timed]"
---

# /cert-drill: targeted CCAR-F practice

Files: `resources/cheat-sheet.md` (source of truth), `my-mistakes.md` / `my-progress.md` (personal logs, gitignored — create on first use).

## Parse `$ARGUMENTS`
- **Topic:**
  - `d1`–`d5`: a domain
  - A scenario keyword: `support`, `codegen`, `research`, `devprod`, `ci`, `extraction`, `conversational`
  - `weak`: the lowest-scoring domains in `my-progress.md`
  - `mixed`: blueprint weights (27 / 18 / 20 / 20 / 15)
- **Count:** default = questions per session in `my-plan.md`, else 8. Max 30.
- **`timed`:** 2 minutes per question, like the real exam. No feedback until the end.
- No arguments → ask the topic with AskUserQuestion. Offer `weak` (recommended), `mixed`, a domain, or a scenario.

## Run
1. Read the matching rows of `resources/cheat-sheet.md`. Skim `## Open` in `my-mistakes.md` so you can target the same concepts.
2. Write **new** exam-style questions — same rules as `/cert-today` step 3: realistic scenario, 4 plausible options built on real anti-patterns, ~1 in 6 multiple-response, varied correct letter, in-scope only. Never reuse `resources/practice-questions.md` verbatim.
3. Ask one at a time with AskUserQuestion (options `A`–`D`, text in `description`).
   - **Untimed:** feedback after each answer — the correct answer, why it's right, why the pick was wrong, the meta-rule number.
   - **Timed:** collect all answers, then review the misses in one block.
4. Log: misses → `## Open` in `my-mistakes.md` (same entry format as `/cert-today`); a row in `my-progress.md` with activity `drill: <topic>`. If `my-plan.md` has a `drill` row for today, mark it `done x/n`.
5. End with the score, the % per domain, and the single most important concept to fix.
