---
name: cert-today
description: Daily CCAR-F (Claude Certified Architect – Foundations) study session. Follows today's row in my-plan.md (learn block, checkpoint, lab, mock, practice, light review) or, with no plan, runs a short recap plus an adaptive scenario quiz, logged to your personal files. Use when the user says "cert-today", "daily session", or wants to study for the Claude certification exam today.
argument-hint: "[optional: domain to override, e.g. 'd3' or 'tool design']"
---

# /cert-today: daily CCAR-F session

Files in this repo:
- `resources/cheat-sheet.md`: 18 meta-rules + per-domain decision tables. The source of truth.
- `resources/practice-questions.md`: static bank of 35. **Never reuse these verbatim when generating.** Only `practice` sessions ask them as-is.
- `resources/study-plan-template.md`: phase structure and the lab descriptions.
- `my-plan.md`: the user's schedule, written by `/cert-setup`. `my-progress.md` / `my-mistakes.md`: personal logs. All three are gitignored; create the logs on first use.

## Steps

### 1. Find today's session
Run `date +%F` for today's date.

**If `my-plan.md` doesn't exist:** say in one line that `/cert-setup` builds a schedule from their exam date (~3 min) and ask whether to run it now or just quiz. If they choose setup, follow `.claude/skills/cert-setup/SKILL.md` and stop here. Otherwise use the no-plan fallback below.

**If it exists:**
- Read the header (exam date, questions per session `n`) and find the schedule row for today. If today isn't a study day, use the next upcoming row and say it's a bonus session.
- **Missed rows** (date before today, empty status): mark them `missed`. If 3 or more `learn` rows are missed, tell the user and offer to re-plan with `/cert-setup` (rebuild from today). Otherwise slide today's focus to the earliest missed `learn` block so no domain is skipped.
- Show a one-line header: `Day X of Y · N days to exam (YYYY-MM-DD) · Today: <session> — <focus>`, plus a 1-line recap of the last `my-progress.md` row.
- `$ARGUMENTS` overrides the focus for today only. Leave the plan row as-is and mark it `moved`.

**No-plan fallback:** the weakest domain = the lowest % among domains with at least 5 answered questions in `my-progress.md`. With no history, start at d1 and advance one domain per session. Run it as a `learn` session with n = 8.

### 2. Run the session for today's type

| Session | What to do |
|---|---|
| `learn` | Recap (step 3), then a quiz of **n** questions (step 4): ~75% on the block's topics, 1 on the weakest domain, 1 reworded re-ask from `## Open` in `my-mistakes.md` if any. |
| `drill` | Short recap of the weakest domain's traps, then n questions as in `/cert-drill weak`. |
| `mistakes` | Run `/cert-mistakes` (reworded re-asks of open entries). |
| `checkpoint` | 30-question mixed, **timed** (60 min), blueprint weights, no feedback until the end. Score by domain against the ≥ 75% target. |
| `lab` | Walk through the matching lab (A config / B SDK / C on paper) from `resources/study-plan-template.md` as a checklist. Help them do it in a scratch repo; don't touch this repo. End with 3 check questions on what they built. |
| `practice` | Ask the static questions from `resources/practice-questions.md` (the range in the plan's focus) one at a time, without showing the folded answer, then a few weak-domain questions if time is left. |
| `mock` | Full **60-question timed mock** (120 min), or 30/60 min if the plan says half mock. Fresh questions, blueprint weights, no feedback until the end, then score by domain. **Mock #1 = go/no-go:** under ~70% → recommend rescheduling (or pushing the target date) and offer `/cert-setup`. |
| `light` | No quiz. Clear the last open mistakes (reworded), list the 5 traps they missed most, point them to reread the cheat sheet, and give a logistics checklist (ID, Pearson VUE system check, quiet room). Tell them to stop by early evening. |

### 3. Recap (learn and drill sessions; about 250 words max)
- 3–5 key rules from the focus rows in `resources/cheat-sheet.md`, one "trap to avoid", and one tiny code/config snippet where relevant (a `tool_choice` JSON, `.claude/rules` frontmatter, a hook).
- Stop and wait for the user to say they're ready.

### 4. Question rules (every quiz)
- Write **new** questions in the exam's style:
  - A realistic production situation from one of the exam scenarios: the 6 in guide v1.0 (support agent, Claude Code codegen, multi-agent research, dev productivity, CI, structured extraction) plus conversational AI architecture. (An 8th, *Agentic AI Tools*, is reported but not publicly documented; don't invent its content.)
  - 4 options. Every distractor is plausible and reflects a real anti-pattern: prompt-only enforcement, over-engineering as a "first step", self-reported confidence or sentiment, invented features, blaming a downstream component, a bigger context window as the fix.
  - About 1 in 6 questions is multiple-response ("choose 2").
  - Stay in scope: respect the cheat sheet's "Out of scope" list. Spread the correct letter evenly across A–D.
- Ask **one at a time** with AskUserQuestion: scenario + question in `question`, options labeled `A`–`D` with the option text in `description`, `multiSelect: true` for multiple-response. Never mark a recommended option.
- Untimed sessions: after each answer, 2–4 lines: ✅/❌, why the correct answer wins, why the pick loses, and the **meta-rule number (1–18)** from the cheat sheet. Timed sessions: collect all answers, then review the misses in one block.

### 5. Log results
- **Mistakes:** append each miss to `## Open` in `my-mistakes.md`: `| M-NNN | topic | the trap | the correct rule | 0/2 |`. A re-ask answered correctly → streak +1; at 2/2, move the entry to `## Mastered` with today's date.
- **Progress:** append to `my-progress.md`: `| YYYY-MM-DD | <session> – <focus> | x/n (D1 a/b, D3 c/d…) | weak concepts |`.
- **Plan:** set today's row status in `my-plan.md` to `done x/n` (or `done` for lab/light).

### 6. Close
3 lines: the score, the one concept to review tonight, and **tomorrow's session from the plan** (date + focus). Suggest `/cert-drill <domain>` if any domain scored below 60%. If recent sessions and the latest mock show ≥ 90% overall and every domain ≥ 75%, tell them they're ready.
