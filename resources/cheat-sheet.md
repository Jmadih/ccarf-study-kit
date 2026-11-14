# CCAR-F Cheat Sheet

> The exam is scenario-based judgment. Most questions come down to a small set of recurring decision rules. Learn the rules, then spot which one a question is testing.

## The meta-rules (the exam's "answer DNA")
1. **Deterministic beats probabilistic** when money, identity, or compliance is at stake → hooks / programmatic gates, not "stronger prompt wording".
2. **Fix the root cause the logs point to.** If the coordinator decomposed badly, don't blame downstream agents.
3. **Proportionate first step.** Prefer the cheap, high-leverage fix (better tool description, explicit criteria + few-shot) over new infrastructure (classifiers, routing layers, fine-tuning).
4. **Self-reported confidence and sentiment are unreliable** proxies for complexity. Use explicit criteria instead. Calibrated field-level confidence (checked against labeled data) is fine for review routing.
5. **Structured > generic.** Structured errors, structured handoffs, structured claim→source mappings, structured case facts.
6. **Never silently swallow failures** (empty result marked as success) and **never kill the whole workflow** for one failure.
7. **Least privilege for tools.** 4–5 tools per agent, scoped to role. Add a narrow cross-role tool only for high-frequency needs.
8. **Isolation for quality.** An independent reviewer instance beats self-review. Subagents / `context: fork` / Explore keep verbose output out of the main context.
9. **Match the mechanism to the latency need.** Batch API for overnight or weekly jobs. Synchronous calls for anything blocking.
10. **Distractors often name non-existent features** (`CLAUDE_HEADLESS`, `--batch`, `--headless`, `--format json`, `.claude/config.json`, an `inherit` flag for subagents, `parallel: true`, a guaranteed Batch completion time). If it sounds made up, it probably is.
11. **Descriptions are the primary selection mechanism** → tool descriptions (inputs, edge cases, "use this vs. X"), not list order or system-prompt keywords. If they're vague or overlap, the model guesses. Trap: "reorder the tools" or "add routing guidance to the prompt" are distractors.
12. **Errors must be signaled in the protocol layer** → `isError: true` + `errorCategory` (business/validation/transient/permission) + `isRetryable` + message. Not buried in result fields like `"error_code"` or text prefixes like "SYSTEM ERROR:". Trap: validation errors in fields that say "success" still read as success.
13. **`tool_choice` controls whether a tool is called**, not which tool. `"auto"` = text-or-tool, `"any"` = must call *some* tool (model picks), `{"type":"tool","name":"X"}` = force X. Trap: using `"any"` when you want a single named tool.
14. **Mode parameters hide selection.** A single `analyze_code` tool with `mode: bugs | performance | style` is harder to select than three separate tools. Split generic tools into purpose-specific ones → descriptions become the selection guide. Trap: "improve the mode description" leaves the hidden choice.
15. **Business errors (policy, compliance, eligibility)** use `errorCategory: business, isRetryable: false`. Validation errors (bad format, missing field) use `validation, isRetryable: true` → invite reformatting. Use the right category to signal intent. Trap: marking a policy refusal as "validation" tells the model to retry with different input.
16. **MCP scope: `.mcp.json` (project, shared, committed) vs. `~/.claude.json` (user, personal).** Both support `${ENV_VAR}` expansion. All MCP servers' tools are available at the same time. Trap: hardcoding servers into CLAUDE.md or the prompt; putting team config in user config.
17. Glob = find by name. Grep = search inside. Use them sequentially, not combined.
18. **Separate lookup from action.** A read-only search tool resolves ambiguity and returns a machine ID; the mutating tool accepts **only that ID** — never ambiguous human text (team nicknames, reused flight numbers, codeshares). A wrong-target mutation *succeeds silently*; format validation can't catch it.

---

## Domain 1: Agentic architecture & orchestration (27%)
| Topic | Remember |
|---|---|
| Agentic loop | Continue while `stop_reason == "tool_use"`, stop on `"end_turn"`. Append tool results to history each iteration. |
| Tool results | The **harness** runs the tool and appends a `tool_result` block that references the `tool_use` id, then calls the model again. The API never runs your tools or pushes results itself. |
| Loop anti-patterns | Parsing text like "I'm done", using an iteration cap as the *primary* stop, treating "has text content" as done. |
| Hub-and-spoke | Coordinator routes **all** communication, error handling, and aggregation. Subagents don't talk to each other. |
| Subagent context | **No inheritance.** Pass complete prior findings explicitly in the prompt. Use structured data to separate content from metadata (URL, doc, page). |
| Spawning | `Task` tool; the coordinator's `allowedTools` must include `"Task"`. Parallel = **multiple Task calls in one response**. |
| "Coordinator never delegates" | Its `allowedTools` doesn't include `"Task"`. Configuration beats instructions. Don't blame the model tier or vague subagent descriptions. |
| Subagent facts | No inheritance, no `inherit` flag, no memory between invocations. Tool restrictions apply to **built-in and MCP tools alike**. |
| AgentDefinition | description + system prompt + tool restrictions per subagent type. |
| Coordinator prompts | State goals and quality criteria, not step-by-step procedures. |
| Narrow decomposition | Classic failure: "creative industries" split only into visual arts. The coordinator is at fault. |
| Iterative refinement | Coordinator checks synthesis for gaps → re-delegates targeted queries → re-synthesizes. |
| Detection ≠ repair | A reviewer that only scores the report and ships it anyway changes nothing. Re-delegating to fill the gaps is the loop. |
| Enforcement | Prerequisite gates (block `process_refund` until `get_customer` returns a verified ID). |
| Hooks | `PostToolUse` = transform/normalize results (timestamps, status codes, trim fields). Pre-call interception = block policy violations (refund > $500 → escalate). |
| Handoff to human | Structured summary: customer ID, root cause, amount, recommended action. The human has no transcript. |
| Multi-concern message | Split it into separate items, investigate each against the shared customer context, then give **one unified answer**. Don't handle only the first, ask the customer to pick one, or escalate. |
| Decomposition | **Prompt chaining** for predictable multi-aspect work. **Dynamic/adaptive** for open-ended investigation (map → prioritize → adapt). |
| Sessions | `--resume <name>` when prior context is mostly valid (`--continue` = the *most recent* session in the directory, not a named one). `fork_session` to branch divergent approaches from a shared baseline. **Fresh session + structured summary** when tool results are stale. On resume, tell the agent exactly which files changed. |
| Stale sessions | After big changes to the code, start **fresh with a summary**. `/compact` on stale results just compresses wrong information. Re-reading files one by one mixes old and new states (OK only for small changes). |

## Domain 2: Tool design & MCP (18%)
| Topic | Remember |
|---|---|
| Descriptions | The **primary** selection mechanism. Include inputs, example queries, edge cases, and "use this vs. X". |
| Overlap | Rename and differentiate (`analyze_content` → `extract_web_results`). Split generic tools into purpose-specific contracts. |
| Mode parameters | A single tool with a `mode` parameter (e.g. analyze_document) hides the selection problem. **Split it** into purpose-specific tools. Merging tools behind a mode parameter is a distractor. |
| Selection non-fixes | Reordering the tool list, moving selection guidance from tool descriptions into the system prompt, or adding a "routing tool" that picks tools: all distractors. |
| System prompt | Keyword-sensitive instructions can override good tool descriptions. Audit them. |
| Errors | `isError: true` + `errorCategory` (transient / validation / business / permission) + `isRetryable` + human-readable message. |
| Business errors | Policy refusal = `errorCategory: business`, `isRetryable: false` + a **customer-friendly explanation**. Not `validation` (that invites reformulating the request), and don't pass internal rule IDs. Don't flatten every failure into one status like "search unavailable". |
| isError vs text | A prefix like "SYSTEM ERROR:", or a JSON `error` field inside a *successful* result, still reads as success. Use the protocol's **`isError`** flag. |
| Empty ≠ failure | A valid "no matches" is a success. A timeout is an access failure. Report them differently. |
| Local recovery | Subagent retries transient errors itself and propagates only unresolved ones, with partial results and what it tried. |
| `tool_choice` | `"auto"` = may answer in text. `"any"` = must call *some* tool. `{"type":"tool","name":"X"}` = must call X (e.g., force `extract_metadata` first). |
| Lookup vs action | Read-only search returns machine IDs; the mutating tool accepts **only the ID**, never ambiguous text. Wrong-target mutations succeed silently — validation can't fix "which one did you mean". |
| Too many tools | 18 tools degrade selection. Scope to role. |
| MCP scope | `.mcp.json` (project, committed, shared) with `${ENV_VAR}` expansion. `~/.claude.json` (user, personal/experimental). All servers' tools are available at the same time. |
| Resources vs tools | **Resources** = content catalogs (schemas, docs trees, issue lists) that cut down exploratory calls. **Tools** = actions. |
| Resource distractors | A bespoke `describe_available_data` tool, or hard-coding the inventory into the prompt (goes stale), instead of MCP resources. |
| Build vs reuse | Use a community MCP server for standard integrations (Jira). Build custom ones only for team-specific workflows. |
| Built-ins | **Grep** = search inside files. **Glob** = find files by name pattern (`**/*.test.tsx`). **Edit** needs unique anchor text; if it isn't unique, fall back to **Read + Write**. Explore incrementally: Grep for entry points, then Read to follow imports. |
| Built-in distractors | `Bash grep -r` instead of Grep/Glob · Edit with "replace all" when only one occurrence should change · `sed` · deleting duplicates to make the anchor unique. |
| Preview → execute | A `dry_run` flag gets bypassed. Split into two tools: `preview_X` returns the impact **plus a single-use confirmation token**; `execute_X` accepts only that token. The interface enforces the sequence — instructions can't. |
| Tool-level retries | Timeouts → retry inside the tool with backoff. Syntax/validation errors → return immediately with the details. Uniform retry burns time on calls that can never succeed. |

## Domain 3: Claude Code configuration (20%)
| Topic | Remember |
|---|---|
| CLAUDE.md hierarchy | User `~/.claude/CLAUDE.md` (personal, **not shared**) → project (`CLAUDE.md` or `.claude/CLAUDE.md`) → subdirectory CLAUDE.md. |
| "New teammate doesn't get the rule" | The rule lives in user-level config. Move it to the project level. |
| settings.json | Holds permissions and tool settings, **not** standards prose. Copying your user CLAUDE.md to a teammate's machine drifts over time. Verify what loaded with `/memory`. |
| Huge CLAUDE.md | Split it into `.claude/rules/` topic files (path-scoped where relevant). Don't cut it to a vague summary or turn it into opt-in commands. |
| Modular | `@path/to/file.md` imports. `.claude/rules/*.md` topic files. |
| Path rules | `.claude/rules/x.md` with YAML `paths: ["**/*.test.tsx"]`. Loads only for matching files. Better than a subdirectory CLAUDE.md for conventions **spread across directories**. |
| `/memory` | Shows which memory files are loaded. Use it to diagnose inconsistent behavior. |
| Commands | `.claude/commands/` = team, version-controlled. `~/.claude/commands/` = personal. |
| Skills | `.claude/skills/<name>/SKILL.md`. Frontmatter: `context: fork` (isolated subagent, keeps verbose output out of the main session), `allowed-tools` (restricts tools), `argument-hint` (prompts for params). Personal variants go in `~/.claude/skills/` under a **different name**. |
| Frontmatter fit | Verbose output → `context: fork` · guarantee "never runs Bash" → `allowed-tools` (configuration, not instructions) · invoked without arguments → `argument-hint`. A longer description doesn't stop bare invocations. |
| Skill vs CLAUDE.md | Skill = on-demand, task-specific. CLAUDE.md = always loaded, universal standards. |
| Plan mode | Large-scale, multi-file, architectural, several valid approaches (microservices, a migration touching 45+ files). **Direct** for a clear single-file fix. Combine them: plan, then execute. |
| Explore subagent | Isolates verbose discovery and returns a summary. |
| Iteration | Prose misread → give 2–3 concrete I/O examples. TDD: write tests first, share the failures. Interview pattern: have Claude ask questions first in unfamiliar domains. Interacting issues → one message. Independent issues → fix sequentially. |
| CI/CD | `claude -p "..."` (non-interactive). `--output-format json` + `--json-schema` for machine-parseable findings. Put review criteria and test standards in CLAUDE.md. Use an **independent** review instance, not the generating session. On re-review, include prior findings and report only new or unaddressed issues. Give it the existing tests to avoid duplicates. |
| CI distractors | Dedup by hashing file+line (breaks when lines move) · reviewing only the newest diff · one review per PR · a bigger model tier, temperature 0, or "be harsher" for self-review bias (the real problem is shared context). |

## Domain 4: Prompt engineering & structured output (20%)
| Topic | Remember |
|---|---|
| Explicit criteria | "Flag only when the comment contradicts the code" beats "be conservative" or "high-confidence only". |
| Severity consistency | Explicit severity criteria **with code examples per level**. Not majority voting over 3 samples, not severity by line count. |
| False positives | They erode trust in *all* categories. Temporarily disable noisy categories while you fix them. Define severity with code examples. |
| Few-shot | Best tool for consistent format and ambiguous cases. Use 2–4 targeted examples that **show the reasoning** for choosing A over B. Generalizes to new patterns and reduces hallucination in extraction. Fixes **both** format AND judgment (accepted pattern vs. genuine issue) — a schema can only fix format. |
| Few-shot details | Use **real project code** (not abstract placeholders), include examples of the exact output format, and keep the explicit criteria too (examples add to them, they don't replace them). 20+ examples dilute the signal. |
| tool_use + schema | The most reliable way to get structured output. Removes **syntax** errors, not **semantic** ones (totals that don't sum, values in the wrong field). Inconsistent **value conventions** ("50%" vs 0.5, "last quarter") → **normalization rules in the prompt**, applied at extraction time — the only step where the source is in context. Regex/enums can only reject; downstream scripts lack the doc's context. |
| tool_choice for unknown doc types | Several extraction schemas, must always return structured output → **`"any"`** (the model picks the schema). Forcing one named tool welds every document to one schema. |
| Schema design | Make fields **nullable/optional** when the source may lack the info; otherwise the model fabricates. Use enum + `"other"` + detail string, and `"unclear"` for ambiguous cases. |
| Fabrication non-fixes | "Never fabricate" in the prompt, lower temperature, checksum post-validation: none of them fix a required field. **Make it nullable.** |
| Retry with feedback | Send the original doc + the failed output + the specific validation error. **Useless if the info isn't in the source.** |
| Retry distractors | Retrying without the source doc (the model can't re-check it) or without the error (an untargeted retry). |
| Self-check fields | `calculated_total` vs `stated_total`, `conflict_detected`, and `detected_pattern` (to analyze dismissed findings). |
| Batch API | 50% cheaper, up to 24h, **no SLA**, **no multi-turn tool calling**, `custom_id` for correlation. Resubmit only the failures (chunk oversized docs). Refine prompts on a sample first. SLA math: 30h SLA − 24h processing → submit every ≤ 6h (the official guide's example uses 4h windows for margin). |
| Batch facts | Results are matched by **`custom_id`, not position**. No completion guarantee at any hour. Pre-merge checks stay synchronous. Oversized docs also fail synchronously, so **chunk them**. |
| Multi-pass review | Per-file local passes, then a cross-file integration pass (avoids attention dilution and contradictions). A bigger context window **doesn't** fix this. |
| Independent review | A second instance without the generator's reasoning beats self-review or extended thinking. |
| Prefilling | Robotic openings ("Great question!…") → prefill the assistant turn with the first words of the real answer. Not temperature, post-processing, or a prompt ban. |
| Vague requests | Clarifying-question ladders cause abandonment → proceed on **stated assumptions** and invite correction. Not hidden defaults, not one-more-question, not a form. |
| Contradictory preferences | "Keep it budget-friendly" + "I want the VIP seats" → surface the conflict and ask which wins. Never latest-wins, both-answers, or split-the-difference. |
| Instruction drift | Rules followed early, ignored by turn 10+ → re-inject reminders at conversation breakpoints; better, replace verbose rules with few-shot examples. Accumulated assistant text dilutes the system prompt's share of context. |

## Domain 5: Context management & reliability (15%)
| Topic | Remember |
|---|---|
| Summarization risk | Numbers, dates, %, and customer expectations get blurred. Keep a persistent **"case facts" block** outside the summarized history. |
| Lossy-summary non-fixes | Summarizing more often (makes it worse), a bigger `max_tokens`, "only quote from the latest summary". The pair that works: a **case-facts block + trimming tool results**. |
| Lost in the middle | Put the key-findings summary **at the beginning** and use explicit section headers. |
| Tool bloat | Trim to the relevant fields (a 40-field order → 5 fields), e.g., with a PostToolUse hook. |
| Escalate when | The customer **explicitly asks** (right away, no investigation first), policy is **silent or ambiguous** (competitor price match), or the agent can't make progress. |
| Explicit human request | Escalate **immediately**. Don't even "gather order details for the handoff" first. |
| Don't escalate | For frustration alone on a solvable issue: acknowledge it and offer to resolve; escalate if they ask again. |
| Multiple matches | Ask for another identifier. Never pick one heuristically. |
| Error propagation | Include failure type, attempted query, partial results, and alternatives. Annotate coverage gaps in the final report. |
| Long exploration | Degradation shows up as vague answers about "typical patterns". Fixes: scratchpad files, subagent delegation, summarizing between phases, `/compact`. |
| Exploration non-fixes | A bigger context window, re-pasting everything into the conversation, `/rewind` (loses discoveries), `/memory` (only diagnoses). |
| Crash recovery | Each agent exports state to a known location. The coordinator loads a **manifest** on resume. |
| Human review | 97% overall accuracy can hide weak segments. Measure accuracy **by doc type and field**. Use **stratified random sampling** of high-confidence items. Calibrate field-level confidence against **labeled validation sets**. Route low-confidence or conflicting items to humans. |
| Review routing | Random 5% sampling *measures* error rates, it doesn't *route* reviews. Self-flagged uncertainty misses errors the model is confidently wrong about. Ranking by document length is a proxy. Don't remove the dashboard after cutting reviews. |
| Provenance | Keep claim → source mappings (URL, doc, excerpt, **date**) all the way through synthesis. With conflicting credible stats, **keep both with attribution** and include **dates** (the figures often measure different moments). Never average them, pick the most recent, or keep only the "more credible" one. Separate established findings from contested ones. A bibliography at the end ≠ claim→source mapping. Render by type (financials → tables, news → prose). |
| Stateless API | The Messages API keeps **no session state** — every request must carry the full `messages` history. "It forgot my earlier turn" = your app dropped the history. No `session_id` parameter exists; a vector DB isn't the fix at conversational scale. |
| Long-conversation memory | Hybrid: critical facts as structured data, older discussion summarized, recent turns verbatim. A bigger window or plain truncation loses the facts that matter. |
| Months-long recall | "What did we conclude about X?" over months of history → semantic search over stored exchanges. Rolling windows discard it; summaries abstract it away. |
| Live events mid-chat | A webhook arrives mid-conversation (order shipped) → prefix it onto the **next user message**. Not the system prompt, not a synthetic user message, not a forced tool call. |

## More distractor patterns
- **"Lower the temperature / temperature 0"** gives deterministic sampling, not binding policy. It never enforces a rule or fixes fabrication.
- **Escalate "to be safe"** when the work is within the agent's ability: partial-failure handling is the agent's job.
- **Measure without acting:** logging failures for later analysis gives the agent no signal *when it makes the decision*.
- **Good vs. best:** expect two reasonable options. The best one fixes the mechanism; the good one only improves the odds.
- **A `mode:` parameter in any tool** → the answer is "split into purpose-specific tools." A wrong-mode call returns valid data for the wrong question — silent failure, no prompt can save it. (Survived 3 misses to learn this one.)
- **Self-reported confidence flags** are produced by the same process that made the error — asking "are you sure?" reuses the judgment that failed. Only calibration against labeled data makes a confidence number mean anything.
- Exam shape: 4 scenarios × ~15 questions, drawn from **8 reported scenarios** — the original 6 plus *Conversational AI Architecture Patterns* and *Agentic AI Tools* (added after guide v1.0; the latter isn't publicly documented yet — watch the community guide). Aim for **≥ 75% in every domain**.

## Out of scope (don't over-study)
Fine-tuning, auth/billing, MCP hosting/infra, model internals, RLHF, embeddings/vector DBs, computer use, vision, streaming/SSE, rate limits/pricing, OAuth, cloud providers, benchmarks, prompt caching details, tokenization.

## Heads-up on naming
The exam guide calls the subagent spawner the **`Task` tool**. Newer Claude Code docs call it the **Agent** tool. Answer in the exam's terms.

The guide also references **`--json-schema`** (paired with `-p --output-format json`), **named sessions** (`--resume <name>`), and **`fork_session`**. Current Claude Code builds have all three: `--json-schema <schema>`, `--resume` by session ID *or* name (set with `-n/--name`), and `--fork-session` (CLI) / `fork_session` (Agent SDK). Older installs may lack some of them — check `claude --help` — but answer in the exam's terms.

## Sources
- Official CCAR-F exam guide (v1.0, July 2026) — via https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request
- Community study guide: https://github.com/paullarionov/claude-certified-architect
- Learning modules: https://claudecertificationguide.com/learn

---
*Part of the [CCAR-F Study Kit](../README.md) · [CC BY 4.0](../LICENSE.md)*
