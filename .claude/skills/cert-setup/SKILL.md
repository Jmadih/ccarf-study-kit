---
name: cert-setup
description: Onboarding for the CCAR-F study kit. Asks for the exam date, study days, daily time budget and background, optionally runs a short diagnostic, then writes a dated study schedule to my-plan.md that every later session follows. Use when the user says "cert-setup", "set up my plan", "onboard me", "I booked my exam", "reschedule", "change my exam date", or starts studying with no my-plan.md yet.
argument-hint: "[optional: exam date YYYY-MM-DD | reschedule]"
---

# /cert-setup: onboarding and study plan

Creates or rebuilds `my-plan.md` (personal, gitignored). Every other session (`/cert-today`, `/cert-drill`, `/cert-mistakes`, and the plain-language sessions in `CLAUDE.md`) reads it to decide what to do each day.

Get today's date with `date +%F` before doing any date math. Never guess it.

## 0. Existing plan?
If `my-plan.md` exists, show the exam date, days left, and how many schedule rows are done, then ask (AskUserQuestion): **Change the exam date** / **Change my study budget** / **Rebuild the plan from today** / **Keep it**. Rebuilding keeps the `done` rows as history and regenerates only rows from today on. Never touch `my-progress.md` or `my-mistakes.md`.

## 1. Ask (one AskUserQuestion call, four questions)
If `$ARGUMENTS` contains a date, skip the first question.

1. **Exam date** (header `Exam date`). Options: `Booked: I'll type the date` / `Not booked: target ~5 weeks` / `Not booked: target ~3 weeks` / `Not booked: target ~8 weeks`. If they picked "booked" or typed something in Other, make sure you have a real `YYYY-MM-DD`; ask again in plain text if not. Reject dates in the past.
2. **Study days** (header `Days`). `Every day` / `Weekdays only` / `Weekends + 2 weekday evenings` / Other (they list the days).
3. **Daily time** (header `Time/day`). `~20 min` / `~40 min` / `~60 min` / `90+ min`.
4. **Background** (header `Background`). `New to Claude API & Claude Code` / `Used one of them` / `Build with both regularly`.

Then one more question: **Run a 10-question diagnostic now?** (`Yes, ~15 min (recommended)` / `Skip, start from D1`). The diagnostic sets your starting point so the plan skips what you already know.

## 2. Diagnostic (if chosen)
10 original mixed questions, one at a time, by blueprint weight: D1 ×3, D3 ×2, D4 ×2, D2 ×2, D5 ×1. Same question rules as `/cert-today` step 3, but **no feedback until the end**. Then show the score per domain, log misses to `## Open` in `my-mistakes.md`, and add a `| date | diagnostic | x/10 (D1 a/3 …) | weak concepts |` row to `my-progress.md`.

## 3. Derive the settings
- **Study days left (N)** = the chosen weekdays from tomorrow (or today, if the user wants to start now) up to the day **before** the exam.
- **Questions per daily session:** 20 min → 6 · 40 min → 10 · 60 min → 15 · 90+ min → 20.
- **Full mocks need a 2-hour block** (60 questions, 120 min). If the daily budget is under 90 min, tell the user to block 2 hours on mock days, or offer a half mock (30 questions, 60 min) instead and note it in the plan.

## 4. Build the schedule
Work **backward from the exam date**, filling study days only:

| Slot | Session | When |
|---|---|---|
| Light review | `light` — clear the mistake log, reread the cheat sheet, logistics. No new material, no timed tests. | Last study day before the exam |
| Mock #2 | `mock` — fresh 60-question timed mock, scored by domain | ~2 study days before |
| Practice | `practice` — the 35 questions in `resources/practice-questions.md` (Q1–18, then Q19–35) + weak-domain drills | Days between the two mocks |
| Mock #1 + go/no-go | `mock` — 60-question timed mock. **Under ~70% → recommend rescheduling** (or, if not booked, push the target date) | ~5 study days before |
| Labs | `lab` — the hands-on labs A/B/C in `resources/study-plan-template.md`. New → 3 lab days · Used one → 2 (A+B) · Builds regularly → 1 (A only, B and C as a checklist) | Before mock #1 |
| Checkpoint | `checkpoint` — 30-question mixed timed drill, target ≥ 75% in every domain | Before the labs |
| Learn | `learn` — one domain block per day: recap + quiz on that block | Everything earlier |

**Learn blocks** follow the Phase 1 table in `resources/study-plan-template.md` (D1 ×4, D2 ×2, D3 ×2, D4 ×2, D5 ×2 = 12 blocks, each with its topics). Fit them to the days available:
- Every 7th study day is `mistakes` (reworded re-asks only, ~20 min) once there are open mistakes.
- **Diagnostic adjusts it:** a domain scored 100% on the diagnostic gets one block fewer (min 1); a domain at 0% gets one extra.
- **Too many days:** after the 12 blocks, add a second pass as `drill` days on the weakest domains, then keep spare days as `drill weak`.
- **Too few days:**
  - Under 25 study days: merge pairs of learn blocks (D2, D3, D4, D5 become one day each; D1 two days).
  - Under 14: drop the labs to one checklist day and drop mock #2.
  - Under 7: skip learn blocks entirely and make every day a `drill weak` + one mock. Tell the user plainly the plan is tight and suggest moving the date if they can.

## 5. Write `my-plan.md`
```markdown
# My CCAR-F plan

- **Exam date:** 2026-12-18 (booked)          ← or "(target, not booked)"
- **Created:** 2026-11-14 · **Last re-planned:** 2026-11-14
- **Study days:** Mon–Fri · **Daily time:** ~40 min → **10 questions/session**
- **Background:** used one of them → 2 lab days
- **Diagnostic:** 6/10 — D1 2/3 · D2 1/2 · D3 2/2 · D4 1/2 · D5 0/1
- **Mock days:** need a 2-hour block

## Schedule
| Date | Phase | Session | Focus | Status |
|---|---|---|---|---|
| 2026-11-16 | Learn | learn | D1: agentic loop & `stop_reason`, hub-and-spoke | |
| … | | | | |
| 2026-12-12 | Practice | mock | Mock #1 + go/no-go (≥ 70%) | |
| 2026-12-17 | Final | light | Clear mistakes, reread cheat sheet, ID + system check | |
```
`Status` is empty until a session fills it with `done x/n`, `missed`, or `moved`.

## 6. Close
Show the first 5 schedule rows and these lines:
- Days left and number of study days.
- What happens next: "Start each day with `/cert-today` (or say *daily session*) and it'll run that day's session."
- "Run `/cert-setup` again any time to change the date or budget."
- If not booked: when to book. That's after mock #1 if it scores ≥ 70%, at the latest.
