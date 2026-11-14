# CCAR-F Study Plan Template

A study-plan structure that fits ~4–6 weeks at **about 1 hour a day**, with buffer days, two timed mock exams, and a go/no-go checkpoint. Work backward from your exam date and map the phases onto your calendar. **Using Claude Code?** `/cert-setup` does this mapping for you: it asks your exam date, study days and daily time, then writes a dated schedule to `my-plan.md` that `/cert-today` follows. Adjust freely — the value is in the *sequence* (rules → drills → mocks) and the mistake log, not the exact dates.

> Exam facts below reflect the **official guide v1.0 (July 2026)** plus community-reported changes through **Nov 2026**. Verify against the [official page](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request) before booking.

## Exam facts
- 60 items (multiple choice and multiple response). **120 min** → about 2 min per item.
- **4 scenarios drawn at random from 8 reported** — the original 6 plus *Conversational AI Architecture Patterns* and *Agentic AI Tools* (added after guide v1.0; the latter isn't publicly documented yet — watch the community guide). You have to know them all.
- **Cost:** free for the first 5,000 Anthropic Partner Network employees; $99 at general availability per the community guide (Nov 2026) — guide v1.0 listed $125, so check the official page when booking. Proctored by Pearson VUE. Valid 12 months, with free renewal.
- Retakes: 14 days after the 1st fail, 30 days after the 2nd, 90 days after the 3rd. Max 4 attempts per 12 months.

| # | Domain | Weight |
|---|---|---|
| 1 | Agentic architecture & orchestration | **27%** |
| 2 | Tool design & MCP integration | 18% |
| 3 | Claude Code configuration & workflows | 20% |
| 4 | Prompt engineering & structured output | 20% |
| 5 | Context management & reliability | 15% |

## The mistake log (the highest-leverage habit)

Keep a numbered log (`M-001`, `M-002`, …). For each missed question, record: the question's topic, the trap, and the correct rule. Then **re-ask missed questions in reworded form** until you answer the underlying rule correctly twice. Misses cluster: in my prep, five or six recurring traps accounted for nearly all errors, and most were "know the rule, fall for the trap wording" — reworded re-asks break exactly that.

## Phase 1 — Learn (one domain block per day, ~1 h)

Read one or two objectives sub-sections per day, in order, and drill 8 questions on what you just covered. Suggested split (D1 has the most content — give it the most days):

| Day | Domain block |
|---|---|
| 1–4 | D1: agentic loop & `stop_reason`, hub-and-spoke, Task tool, context passing, gates & handoffs, hooks, decomposition, sessions |
| 5–6 | D2: tool descriptions, splitting/renaming, error categories, `tool_choice`, MCP scope & resources, built-ins |
| 7–8 | D3: CLAUDE.md hierarchy, `.claude/rules` with `paths:`, commands vs. skills, plan mode, iteration patterns, headless/CI |
| 9–10 | D4: explicit criteria, few-shot, tool_use + schemas, nullable fields, retry with feedback, Batch API, multi-pass review, prefilling, stated assumptions |
| 11–12 | D5: case-facts blocks, lost-in-the-middle, trimming, escalation triggers, scratchpads, sampling & calibration, provenance, stateless API & long-conversation memory |

One light day per week: mistake-log re-asks only (~20 min).

## Phase 2 — Checkpoint + hands-on (~3 days)

1. **Checkpoint:** a timed 30-question mixed drill. Note weak domains — target **≥ 75% in every domain**, not just on average.
2. **Lab day A (config):** in a test repo, set up a project CLAUDE.md, `.claude/rules/` with `paths:`, a skill with `context: fork` + `allowed-tools`, and `.mcp.json` with `${VAR}`. Try plan mode vs. direct execution.
3. **Lab day B (SDK):** a minimal Agent SDK loop — a `stop_reason` loop, 3 tools with structured errors, and a hook that blocks refunds over $500.
4. **Lab day C (on paper):** sketch an extraction schema (nullable, enum + other, calculated vs. stated total) and a multi-agent error/provenance design.

## Phase 3 — Practice (last ~5 days)

- **All 35** [practice questions](practice-questions.md) + the official sample questions. For each, explain why every distractor is wrong.
- **Mock #1** (~5 days before): a full timed 60-question mock. Log the score **by domain**.
- **Go/no-go:** under ~70% → reschedule (free reschedule windows usually close a day or two before the slot — check when booking).
- **Mock #2** (~2 days before): a second full mock on a fresh question set.
- **Day before — light only:** clear the last mistake-log entries, reread the [cheat sheet](cheat-sheet.md), prep logistics (ID, system check). **No new material, no timed tests.** Stop by early evening.

## Exam-day tactics
- For each question, **name the tested rule first**: deterministic vs. prompt, root cause, proportionate first step, isolation, least privilege, latency fit.
- Eliminate options that invent features, add heavy infrastructure (classifiers, routing layers, fine-tuning) as a "first step", or rely on self-reported confidence or sentiment.
- Read **"first step", "most effective", and "most maintainable"** carefully — they change the answer.
- Multiple-response items say how many to pick. **Pick exactly that number.**
- Budget the full 2 min/item even when you're fast — use the surplus to reread trap wording.

---
*Part of the [CCAR-F Study Kit](../README.md) · [CC BY 4.0](../LICENSE.md)*
