# CCAR-F Practice Questions

35 original questions written in the exam's style, grouped by seven scenarios. Answers are folded — click to expand. **Pick an answer first, then name the meta-rule from the [cheat sheet](cheat-sheet.md) that the question is testing.** (Works on GitHub and in Obsidian reading mode.)

---

## Scenario 1: Customer support resolution agent

**Q1.** A customer writes: "I don't want to talk to a bot, get me a human now." Their issue is a simple address change the agent could handle. What should the agent do?
A. Offer to make the address change quickly, and escalate only if the customer insists again
B. Run sentiment analysis and escalate if the score is below the threshold
C. Escalate to a human immediately
D. Look up the account first so the human agent has context, then escalate

<details><summary>Answer</summary>

**C.** An explicit request for a human is honored right away, with no investigation first. "Acknowledge and offer to resolve" (A) applies to *frustration*, not to an explicit demand. D delays the handoff; context goes into the structured handoff summary instead.
</details>

**Q2.** `get_customer` returns three accounts named "Maria Lopez". What should the agent do?
A. Pick the account with the most recent order
B. Ask the customer for another identifier (email, order number)
C. Process the request against all three accounts
D. Escalate to a human

<details><summary>Answer</summary>

**B.** For multiple matches, ask for clarification. Never select heuristically.
</details>

**Q3.** Company policy says refunds above $500 need manager approval. The system prompt says so, but logs show the agent occasionally processing $700 refunds. Best fix?
A. Lower the model temperature
B. Add a hook that intercepts `process_refund` calls above $500, blocks them, and routes to the escalation workflow
C. Put the rule in bold capitals at the top of the system prompt
D. Add few-shot examples of refusing large refunds

<details><summary>Answer</summary>

**B.** Business rules with financial impact need deterministic enforcement. C and D are still probabilistic.
</details>

**Q4.** Your MCP tools return dates as Unix timestamps, ISO 8601, and "MM/DD/YYYY", and the agent sometimes misorders events. Most maintainable fix?
A. A PostToolUse hook that normalizes every date to ISO 8601 before the model sees it
B. Use a larger model
C. Rewrite all backend services to one format
D. A system prompt instruction explaining all three formats

<details><summary>Answer</summary>

**A.** PostToolUse hooks normalize heterogeneous tool output deterministically. C would work but is disproportionate.
</details>

**Q5.** After 30 turns, the agent's summary says the customer "wants a partial refund," but the customer said "$84.50 by Friday." What prevents this?
A. Summarize more often
B. Remind the model in the system prompt to be precise
C. Keep a persistent "case facts" block (amounts, dates, order IDs, stated expectations) that is included in every prompt, outside the summarized history
D. Increase the context window

<details><summary>Answer</summary>

**C.** Progressive summarization blurs numbers and dates, so transactional facts should live in a separate, unsummarized layer.
</details>

**Q6.** `lookup_order` returns 42 fields, and a return decision needs 5 of them. Long sessions are running out of context. Best fix?
A. Switch to a model with a larger context window
B. Use `/compact` after every tool call
C. Ask the model to ignore irrelevant fields
D. Trim tool results to the return-relevant fields before they accumulate in context

<details><summary>Answer</summary>

**D.** Trim verbose tool output at the source (for example, with a PostToolUse hook).
</details>

---

## Scenario 2: Code generation with Claude Code

**Q7.** Your team's testing conventions sit in the lead developer's `~/.claude/CLAUDE.md`. A new hire's generated tests ignore them. Root cause and fix?
A. The conventions are user-scoped. Move them to the project CLAUDE.md or `.claude/rules/` in the repo.
B. The new hire is using the wrong model
C. Add them to the new hire's `~/.claude/commands/`
D. The new hire needs to run `/compact`. Nothing else needs to change.

<details><summary>Answer</summary>

**A.** User-level config isn't shared through version control. `/memory` would show which files are loaded.
</details>

**Q8.** You want a `/analyze-deps` skill that reads hundreds of files and returns only a short report, without flooding your main conversation. Which frontmatter matters most?
A. `allowed-tools: Bash`
B. `model: opus`
C. `argument-hint`
D. `context: fork`

<details><summary>Answer</summary>

**D.** `context: fork` runs the skill in an isolated subagent context so verbose output stays out of the main session.
</details>

**Q9.** You need to fix a null-pointer bug. The stack trace points to one line in one file. Which approach?
A. Plan mode
B. Spawn an Explore subagent first
C. Write an ADR first
D. Direct execution

<details><summary>Answer</summary>

**D.** The change is clear and single-file, so plan mode would be overhead.
</details>

**Q10.** Your prose description of a date-format transformation keeps being interpreted inconsistently. What's the most effective next step?
A. Provide 2–3 concrete input → output examples
B. Switch to plan mode
C. Ask Claude to be more careful
D. Rewrite the description in more detail

<details><summary>Answer</summary>

**A.** Concrete I/O examples beat prose when prose is being read inconsistently.
</details>

**Q11.** You're about to implement caching in a domain you don't know well. Which technique surfaces design issues (invalidation, failure modes) before any code is written?
A. Direct execution with detailed instructions
B. The interview pattern: ask Claude to question you first
C. Few-shot examples
D. Test-driven iteration

<details><summary>Answer</summary>

**B.** The interview pattern surfaces considerations you may not have anticipated.
</details>

---

## Scenario 3: Multi-agent research system

**Q12.** The synthesis subagent writes a report that ignores the web search findings entirely. The coordinator called it with "Synthesize the research findings." Root cause?
A. The web search agent returned malformed output
B. The synthesis agent needs web search tools
C. Subagents don't inherit the coordinator's context, so the findings must be passed explicitly in the prompt
D. The synthesis model is too small

<details><summary>Answer</summary>

**C.** Subagents have no automatic context inheritance.
</details>

**Q13.** Three independent research subtasks run one after another, which triples latency. Fix?
A. Use the Batch API
B. Raise `max_tokens`
C. Merge the three subagents into one
D. Emit multiple Task tool calls in a single coordinator response

<details><summary>Answer</summary>

**D.** Parallel spawning means multiple Task calls in one response. The Batch API has no latency SLA.
</details>

**Q14.** Two credible sources report 34% and 41% adoption for the same metric. What should the synthesis do?
A. Pick the more recent source
B. Omit the statistic
C. Report both values with attribution and dates, and flag the conflict
D. Average them to 37.5%

<details><summary>Answer</summary>

**C.** Annotate conflicts with source attribution. Dates may also explain the difference (temporal, not contradictory).
</details>

**Q15.** Final reports lose track of which claim came from which source. Where is the fix?
A. Ask the report agent to add citations
B. Increase the context window
C. Run a post-hoc citation search
D. Have subagents output structured claim → source mappings (URL, excerpt, date) that synthesis must preserve

<details><summary>Answer</summary>

**D.** Attribution is lost at summarization steps, so provenance has to be carried as structured data from upstream.
</details>

**Q16.** The coordinator has 18 tools, including web search, PDF parsing, and report formatting, and it often picks the wrong one. Best approach?
A. Better tool descriptions only
B. Set `tool_choice: "any"`
C. Distribute tools to specialized subagents (4–5 each) and give the coordinator mainly Task
D. Add a routing classifier

<details><summary>Answer</summary>

**C.** Too many tools degrade selection. Scope tools by role.
</details>

---

## Scenario 4: Developer productivity

**Q17.** You need to find every file matching `*.controller.ts` across the repo. Which tool?
A. Read
B. Bash `cat`
C. Glob
D. Grep

<details><summary>Answer</summary>

**C.** Glob matches file paths. Grep searches file *contents*.
</details>

**Q18.** `Edit` fails because the anchor text `return null;` appears 9 times in the file. What next?
A. Read the full file, then Write the modified version (or use a larger unique anchor)
B. Retry Edit
C. Delete the file and regenerate it
D. Use Grep to edit

<details><summary>Answer</summary>

**A.** Read + Write is the documented fallback when there's no unique match.
</details>

**Q19.** Your team wants Jira integration. What should you do?
A. Put it in each developer's `~/.claude.json`
B. Build a custom MCP server
C. Hardcode the token in `.mcp.json`
D. Use an existing community Jira MCP server, configured in project `.mcp.json` with `${JIRA_TOKEN}`

<details><summary>Answer</summary>

**D.** Reuse community servers for standard integrations, use project scope for shared tooling, and use env var expansion for secrets.
</details>

**Q20.** The agent keeps using Grep to search your database schema even though you built a much richer MCP `query_schema` tool. First fix?
A. Remove Grep
B. Improve the MCP tool's description to explain its capabilities and outputs, and/or expose the schema as an MCP resource
C. Tell users to mention the tool name
D. Use `tool_choice` to force `query_schema` on every turn

<details><summary>Answer</summary>

**B.** Weak descriptions make agents fall back to built-ins. Resources expose catalogs without exploratory calls.
</details>

**Q21.** After two hours exploring a legacy codebase, the agent starts talking about "typical MVC patterns" instead of the specific classes it found earlier. Best mitigation?
A. Start over from scratch
B. Lower temperature
C. Keep a scratchpad file of key findings, reference it, and delegate further exploration to subagents (plus `/compact`)
D. Use a larger model

<details><summary>Answer</summary>

**C.** These are the standard fixes for context degradation.
</details>

---

## Scenario 5: Claude Code for CI

**Q22.** Your automated PR review must post inline comments, which requires machine-parseable output. Which flags?
A. `--batch --json`
B. `-p --output-format json --json-schema <schema>`
C. `CLAUDE_OUTPUT=json`
D. `-p` only, then regex the text

<details><summary>Answer</summary>

**B.** Non-interactive print mode plus JSON output constrained by a schema gives findings your CI script can parse reliably. `--batch` and `CLAUDE_OUTPUT` (A, C) don't exist; regexing free text (D) breaks on the first wording change.
</details>

**Q23.** After each new commit, the review re-posts the same 12 comments. Fix?
A. Raise temperature for variety
B. Include the prior findings in context and tell it to report only new or still-unaddressed issues
C. Deduplicate comments with string matching
D. Run the review only once per PR

<details><summary>Answer</summary>

**B.** The reviewer needs to know what it already said. String-matching dedup (C) breaks as soon as wording or line numbers shift; one review per PR (D) misses new issues; temperature (A) only changes the wording.
</details>

**Q24.** Developers ignore the bot because the "documentation accuracy" category is ~60% false positives. What's the best immediate action?
A. Add "only report high-confidence findings" to the prompt
B. Require three runs to agree
C. Switch models
D. Temporarily disable that category while rewriting its criteria to be explicit (e.g., "flag only when the comment contradicts the code's behavior")

<details><summary>Answer</summary>

**D.** Vague confidence filters don't raise precision. Disabling a noisy category restores trust, and explicit criteria fix it.
</details>

**Q25.** The same session that generated a feature is asked to review it and finds nothing. Why, and what's the fix?
A. Add "be critical" to the prompt
B. Enable extended thinking
C. The session keeps its own reasoning context and rarely questions it. Use an independent review instance.
D. The session hit its token limit. Use `/compact`.

<details><summary>Answer</summary>

**C.** Self-review bias comes from shared context, not effort or budget. Extended thinking (B) and "be critical" (A) still reason inside the same context; `/compact` (D) doesn't address the bias.
</details>

---

## Scenario 6: Structured data extraction

**Q26.** Invoices sometimes lack a PO number, and the model invents plausible ones. Fix?
A. Add retries until a PO number appears
B. Add "don't hallucinate" to the prompt
C. Use `tool_choice: "any"`
D. Make `po_number` nullable/optional in the schema and instruct the model to return null when it's absent

<details><summary>Answer</summary>

**D.** Required fields push the model to fabricate. Retries can't recover information that isn't there.
</details>

**Q27.** Extraction output is always valid JSON, but line items sometimes don't sum to the total. What's going on and what do you do?
A. This is a semantic error that tool use can't prevent. Extract `calculated_total` alongside `stated_total`, validate, and retry with the specific error.
B. This is a syntax error. Use strict mode.
C. Lower temperature
D. Switch to a free-text response

<details><summary>Answer</summary>

**A.** Schemas remove syntax errors, not semantic ones. Strict mode (B) only guarantees shape; free text (D) loses structure; temperature (C) doesn't make arithmetic consistent.
</details>

**Q28.** You don't know whether each document is an invoice, receipt, or contract, and you have one extraction tool per type. You need structured output every time. Which setting?
A. Force `extract_invoice`
B. `tool_choice: "any"`
C. `tool_choice: "auto"`
D. No tools. Ask for JSON in the prompt.

<details><summary>Answer</summary>

**B.** The model must call *a* tool but chooses which one.
</details>

**Q29.** Nightly processing of 10,000 documents, with results due the next business day. What's the most cost-effective approach?
A. Batch API with multi-turn tool calls for validation
B. Message Batches API with `custom_id`, resubmitting only failures (chunking oversized docs)
C. Synchronous API with parallelism
D. Claude Code `-p` in a loop

<details><summary>Answer</summary>

**B.** A is wrong because the Batch API doesn't support multi-turn tool calling.
</details>

**Q30.** Overall extraction accuracy is 97%, and management wants to remove human review. What first?
A. Remove review for everything with confidence > 0.9
B. Accept it, since 97% is above target
C. Analyze accuracy by document type and field, and use stratified sampling of high-confidence items, before reducing review
D. Ask the model for its own accuracy estimate

<details><summary>Answer</summary>

**C.** Aggregate metrics can hide weak segments.
</details>

---

## Scenario 7: Conversational AI architecture

**Q31.** A user tells your cooking assistant "I'm allergic to peanuts." Six messages later, it suggests a satay recipe. The whole conversation is well under the context window. Most likely cause?
A. The app isn't sending the earlier turns in the `messages` array on each request
B. The API requires a `session_id` parameter to link turns
C. Conversation memory requires a vector database
D. The conversation exceeded the model's context window

<details><summary>Answer</summary>

**A.** The API is stateless — turns that aren't in `messages` never happened. A six-message exchange can't exceed the window, and no `session_id` parameter exists.
</details>

**Q32.** Users complain your writing assistant opens every reply with "Great question! Here's what I found…" Most effective fix?
A. Prefill the assistant turn with the first words of the real answer
B. Add "never open with a greeting" to the system prompt
C. Post-process responses to strip opening phrases
D. Lower the temperature

<details><summary>Answer</summary>

**A.** Prefilling prevents the pattern at generation time — the model continues from the prefill. Prompt bans (B) and post-processing (C) fight the symptom; temperature (D) controls randomness, not phrase choice.
</details>

**Q33.** While planning a trip, the user says "keep it budget-friendly," then later "actually, I want the VIP seats for the show." They now ask you to finalize the itinerary. What should the assistant do?
A. Produce two itineraries, one per constraint
B. Point out the conflict and ask which priority wins
C. Build a mid-range itinerary that hedges between the two
D. Follow the most recently stated preference

<details><summary>Answer</summary>

**B.** The constraints are incompatible, so choosing for the user — latest-wins (D), splitting the difference (C), or answering both (A) — bakes in an assumption they never made.
</details>

**Q34.** Your `remove_guest_user` tool has a `dry_run` parameter for previewing impact, but logs show the agent calling with `dry_run=false` immediately. You must guarantee every removal is previewed and confirmed. Best design?
A. Split into two tools: `preview_remove_guest` returns the impact plus a single-use confirmation token; `execute_remove_guest` requires that token
B. Instruct the agent to always call with `dry_run=true` first and wait for confirmation, with few-shot examples
C. A server-side rule allowing `dry_run=false` only within 60 seconds of an identical `dry_run=true` call
D. Route the tool through an orchestration layer that asks the user to approve every call

<details><summary>Answer</summary>

**A.** The token binds execution to a completed preview at the interface level — architecture, not instructions (B) or timing heuristics (C). D adds an approval gate but doesn't tie execution to the preview.
</details>

**Q35.** Users of your assistant ask vague things like "make this better," get four clarifying questions in reply, and 40% abandon the session. Best approach?
A. Proceed on reasonable assumptions, state them explicitly, and invite correction
B. Direct users to a structured intake form
C. Ask all clarifying questions in one compound message
D. Proceed with hidden defaults to keep the reply short

<details><summary>Answer</summary>

**A.** Stated assumptions give an immediate useful answer while keeping the user in control. Hidden defaults (D) misfire silently; more questions (C) or a form (B) add friction, which is the problem.
</details>

---

## Scoring
- **31+/35:** ready. Take the official practice test and a full mock exam.
- **25–30:** review the domains you missed in the [cheat sheet](cheat-sheet.md).
- **<25:** go through the domain modules at [claudecertificationguide.com/learn](https://claudecertificationguide.com/learn) again.

---
*Part of the [CCAR-F Study Kit](../README.md) · [CC BY 4.0](../LICENSE.md) · All questions are original — no actual exam content.*
