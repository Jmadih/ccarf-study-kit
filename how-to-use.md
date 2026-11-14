# How to Use This Kit

Three ways to study with it — pick one, or mix. The files are plain Markdown; nothing here requires an account or a setup wizard.

| Path | Best if you want… |
|---|---|
| [Claude Code](#option-1--claude-code-the-kit-becomes-a-tutor) | An interactive tutor that quizzes you, tracks your mistakes, and re-asks them until they stick |
| [Obsidian](#option-2--obsidian) | Notes inside your own vault, folded answers, your own mistake log |
| [Plain Markdown / GitHub](#option-3--plain-markdown) | Zero setup — read in the browser, print the cheat sheet |

---

## Option 1 — Claude Code (the kit becomes a tutor)

The repo ships with a `CLAUDE.md`, which Claude Code reads automatically when you start it in this folder. That file turns the session into a **CCAR-F quiz tutor** — no prompting wizardry needed.

1. **Install Claude Code** — https://claude.com/claude-code (npm or native install; any plan works, including Pro/Max subscriptions)
2. **Clone or download this repo**, then open a terminal in the folder:
   ```bash
   git clone https://github.com/Jmadih/ccarf-study-kit.git   # or use GitHub's "Download ZIP"
   cd ccarf-study-kit
   claude
   ```
3. **Onboard once: `/cert-setup`** (or just say `set up my plan`). It asks four things — **exam date** (or a target date if you haven't booked), **which days you study**, **how much time per day**, and **your background** — then offers a 10-question diagnostic. From that it writes `my-plan.md`, a dated schedule worked backward from your exam:

   | Your answer | What it configures |
   |---|---|
   | Exam date | The whole calendar: learn blocks → checkpoint → labs → mock #1 (go/no-go) → practice → mock #2 → a light last day |
   | Study days | Which dates get sessions; days in between are skipped, not crammed |
   | Time per day | Questions per session: ~20 min → 6 · ~40 → 10 · ~60 → 15 · 90+ → 20. Mock days need a 2-hour block (or a half mock) |
   | Background | Number of hands-on lab days (3 if new, 1 if you build with Claude daily) |
   | Diagnostic | Extra learn days for weak domains, fewer for ones you already know |

   Short on time? Under 25 study days the learn blocks merge; under 14 the labs shrink and mock #2 is dropped; under a week it's straight weak-spot drills — and it'll tell you honestly if the date looks too tight. Changed your exam date or missed a few days? Run `/cert-setup` again — it keeps your history and rebuilds the plan from today.
4. **Then just talk to it, every day.** The commands below are conventions, not strict syntax — any phrasing with the same intent works:

   | Say | What happens |
   |---|---|
   | `daily session` *(or `quiz me`, `study`)* | Days-to-exam header, a one-line recap, then **today's session from your plan** — a learn block, a drill, a checkpoint, a lab, a mock, or the light final day. Without a plan: 8 adaptive questions on your least-seen or weakest domains |
   | `drill d3` · `drill tool design 10` | Targeted practice: **n questions on one domain** (default: your plan's per-session count, else 8) — by number (`d1`–`d5`) or by topic name |
   | `drill weak 10` | Questions drawn from your **weakest domains**, per your progress log |
   | `review my mistakes` | Spaced re-asks of your **open** mistakes — same rules, reworded scenarios. A correct answer advances the streak; **two in a row = mastered**, and the entry is never asked again |
   | `explain meta-rule 14` · `explain the batch api` | Teaches it from the cheat sheet with a fresh original example, then one check question to confirm it landed |

   The repo also bundles these as real slash commands (skills in `.claude/skills/`): **`/cert-setup`**, **`/cert-today`**, **`/cert-drill <topic> <n> [timed]`** (timed = 2 min per question, feedback at the end), and **`/cert-mistakes`** — they appear in Claude Code's `/` menu automatically when you open this folder.

   Domain shorthand: `d1` agentic architecture (27%) · `d2` tool design & MCP (18%) · `d3` Claude Code config (20%) · `d4` prompting & structured output (20%) · `d5` context & reliability (15%)

**What the tutor does:**
- Asks **one question at a time** and never reveals the answer until you commit to a choice
- After each answer: correct/incorrect, why the distractors are wrong, and the **meta-rule number** from the cheat sheet
- Writes every miss to `my-mistakes.md` (trap + correct rule) and each session to `my-progress.md`
- **Re-asks missed rules in reworded form** until you get them right twice in a row — the single highest-leverage habit in this method
- Generates **fresh variant questions** in the exam's style, so the 35 in `resources/practice-questions.md` stay clean for self-testing

Your `my-*.md` files (`my-plan.md`, `my-progress.md`, `my-mistakes.md`) are personal — they're in `.gitignore`, so you can keep the repo cloned in place, commit nothing, and still `git pull` for updates.

> **Note for Obsidian + Claude Code users:** you can point Claude Code at your vault instead and it can quiz you from the notes there. Keeping a separate clone of this repo is simpler: tutor mode activates automatically via the `CLAUDE.md`.

---

## Option 2 — Obsidian

1. **Copy the three files in `resources/`** into your vault — a dedicated folder works well:
   ```
   YourVault/CCAR-F/
   ├── cheat-sheet.md
   ├── practice-questions.md
   └── study-plan-template.md
   ```
2. **Answers fold in Reading View** (and Live Preview): the `<details>` blocks render as click-to-expand. Answer out loud first, *then* expand.
3. **Keep the cheat sheet open beside the questions.** The drill is: read the question → name the meta-rule being tested → check your pick → expand the answer.
4. **Create your own mistake log.** New note, e.g. `CCAR-F Mistakes.md`:

   | ID | Topic | The trap I fell for | The rule | Streak |
   |----|-------|---------------------|----------|--------|
   | M-001 | Tool selection | Picked "improve the mode description" | Meta-rule 14: split mode-parameter tools | 0/2 |

   Every study session, **re-ask your open entries in reworded form** until each reaches 2/2. The [Spaced Repetition](https://github.com/st3v3nmw/obsidian-spaced-repetition) community plugin works well for this if you'd rather automate it.
5. **Schedule with the template.** Paste the phases from `resources/study-plan-template.md` into a note or your calendar, working backward from your exam date.

---

## Option 3 — Plain Markdown

- Read everything on GitHub — the answers fold there too.
- Print `resources/cheat-sheet.md` (or the PDF export of it) for the final light-review day.
- Track mistakes on paper — the method matters more than the tool.

---

## Whatever you use — the method

1. **Rules first.** Read the cheat sheet before drilling anything.
2. **Name the rule before the options.** For every question, say which meta-rule it tests *before* looking at the answers — that's the exam skill itself.
3. **Log every miss** and re-ask it reworded until it's boring.
4. **Two timed mocks**, scored **by domain** (≥ 75% each). Under ~70% overall → reschedule; it's free until shortly before the slot.

---
*Part of the [CCAR-F Study Kit](README.md) · [CC BY 4.0](LICENSE.md)*
