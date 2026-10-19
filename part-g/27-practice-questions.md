---
title: "27. Practice Questions"
parent: "Part G — Exam Readiness"
nav_order: 27
---

# Chapter 27 — Practice Questions (40)
{: .no_toc }

Scenario-style multiple choice. Pick an answer, then open **Answer** to see the reason and the chapter to revise. The answer key is at the end.
{: .fs-5 .fw-300 }

<details open markdown="block">
<summary>Contents</summary>
{: .text-delta }
1. TOC
{:toc}
</details>

---

## D1 · Agentic Architecture (Q1–11)

**Q1.** ShopAssist sometimes replies *"Let me check that order…"* and then stops. The loop exits as soon as the response contains text. What is the best fix?
- A) Exit when the text contains "done" or "complete"
- B) Branch on `stop_reason`, and keep looping while it is `tool_use`
- C) Set `max_turns=3`
- D) Set `tool_choice` to `any` on every turn

<details markdown="1"><summary>Answer</summary>
**B**: `stop_reason` drives the loop. A turn can contain text **and** a `tool_use` block. → [Ch 5](../part-b/05-agentic-loop.html)
</details>

**Q2.** In 12% of conversations the agent calls `lookup_order` without first verifying the customer with `get_customer`. What is the most effective fix?
- A) Make the verification instruction bold in the system prompt
- B) Add few-shot examples that show verification first
- C) A programmatic gate that blocks `lookup_order` and `process_refund` until `get_customer` has returned a verified ID
- D) Add a routing classifier before the agent

<details markdown="1"><summary>Answer</summary>
**C**: a rule that must always hold needs **deterministic** enforcement. Prompts and examples are probabilistic. → [Ch 8](../part-b/08-reliability-oversight.html)
</details>

**Q3.** A research system on *"AI in creative industries"* returns accurate findings, but only about visual art. The subagent logs show they did exactly what they were asked. What is the root cause?
- A) The search subagent's queries were poor
- B) The synthesis agent dropped the other topics
- C) The context window was too small
- D) The coordinator's decomposition was too narrow

<details markdown="1"><summary>Answer</summary>
**D**: coverage is decided where the work is decomposed. The subagents only cover the subtasks they are given. → [Ch 6](../part-b/06-task-decomposition.html)
</details>

**Q4.** The search subagent times out on one subtopic. How should it report this to the coordinator?
- A) Return an empty result marked as success
- B) Return a structured error: failure type, the query tried, partial results, a suggested alternative
- C) End the whole research workflow
- D) Retry indefinitely until it succeeds

<details markdown="1"><summary>Answer</summary>
**B**: structured context lets the coordinator recover sensibly. Silent empties, total aborts and endless retries are all wrong. → [Ch 7](../part-b/07-multi-agent.html)
</details>

**Q5.** A subagent's findings ignore the user's constraint ("EU data only"), which the user stated early in the coordinator's conversation. Why?
- A) The constraint wasn't included in the subagent's task prompt. Subagents don't inherit the coordinator's context
- B) The subagent's model is too small
- C) The temperature was too high
- D) The coordinator needed more tools

<details markdown="1"><summary>Answer</summary>
**A**: subagents start with a **fresh, isolated** context. Pass the constraints explicitly. → [Ch 7](../part-b/07-multi-agent.html)
</details>

**Q6.** A compliance report runs the same 4 steps every month: extract, check, summarize, format. Which architecture fits best?
- A) An autonomous agent with all the tools
- B) A multi-agent coordinator
- C) A prompt-chaining workflow with gates between the steps
- D) An unbounded evaluator–optimizer loop

<details markdown="1"><summary>Answer</summary>
**C**: fixed, predictable steps call for the **simplest** architecture, a workflow. → [Ch 4](../part-b/04-agents-vs-workflows.html)
</details>

**Q7.** Refunds over $500 must never run without human approval. Where should this rule live?
- A) In the system prompt, in bold
- B) In few-shot examples of refusals
- C) A lower temperature for refund conversations
- D) A PreToolUse hook (or gate) that denies the call and redirects to escalation

<details markdown="1"><summary>Answer</summary>
**D**: a must-never rule needs **code**. The hook's denial reason lets Claude move on to escalation. → [Ch 8](../part-b/08-reliability-oversight.html)
</details>

**Q8.** A customer with a simple tracking question says: *"I want to talk to a human."* What should the agent do?
- A) Answer the tracking question first, because it's simple
- B) Escalate immediately with a structured handoff summary
- C) Run a sentiment check before deciding
- D) Ask why they want a human

<details markdown="1"><summary>Answer</summary>
**B**: an explicit request for a human is honoured **immediately**. → [Ch 8](../part-b/08-reliability-oversight.html)
</details>

**Q9.** The agent escalates many simple cases but misses real policy-gap cases. What is the best fix?
- A) Escalate when the sentiment score is negative
- B) Escalate when the model's self-reported confidence is below 0.7
- C) Explicit escalation criteria with few-shot examples of escalate and don't-escalate cases
- D) Train a separate classifier model before investigating

<details markdown="1"><summary>Answer</summary>
**C**: escalation is a **judgment** call, so it belongs in the prompt with explicit criteria. Sentiment and self-rated confidence are unreliable. → [Ch 8](../part-b/08-reliability-oversight.html)
</details>

**Q10.** The coordinator spawns subagents for 5 independent topics one at a time, which is slow. What is the best fix?
- A) Have the coordinator issue several Task calls in a single response
- B) Use a larger model for the coordinator
- C) Give each subagent more tools
- D) Merge everything into one agent

<details markdown="1"><summary>Answer</summary>
**A**: several Task calls in one response run in **parallel**. → [Ch 7](../part-b/07-multi-agent.html)
</details>

**Q11.** The synthesis agent keeps sending simple fact checks back through the coordinator, which adds a lot of latency. What is the best fix?
- A) Give the synthesizer every search tool
- B) Remove fact-checking from synthesis
- C) Let the subagents message each other directly
- D) Give the synthesizer a narrowly scoped `verify_fact` tool, and keep complex verification going through the coordinator

<details markdown="1"><summary>Answer</summary>
**D**: a small, **scoped** cross-role tool removes the round trips and keeps least privilege. → [Ch 7](../part-b/07-multi-agent.html)
</details>

---

## D2 · Tool Design & MCP (Q12–18)

**Q12.** The agent often calls `get_customer` when `lookup_order` was the right tool. Both have one-line descriptions. What should you do first?
- A) Expand both descriptions: purpose, input format, when **not** to use the tool
- B) Add a routing classifier
- C) Merge them into one tool
- D) Add few-shot examples of tool selection

<details markdown="1"><summary>Answer</summary>
**A**: descriptions are the main routing signal, so improving them costs little and helps most. → [Ch 9](../part-c/09-tool-design.html)
</details>

**Q13.** When the order database times out, `lookup_orders` returns `[]`. What is wrong?
- A) Nothing. Empty means no orders
- B) It should raise an exception and stop the agent
- C) It hides a failure as a valid empty result. It should return `is_error` with a structured error
- D) It should return the string "Error"

<details markdown="1"><summary>Answer</summary>
**C**: empty ≠ error. Claude will wrongly tell the customer they have no orders. → [Ch 10](../part-c/10-tool-errors-results.html)
</details>

**Q14.** A pipeline must **always** get JSON matching one specific schema. Which `tool_choice`?
- A) `auto`
- B) `none`
- C) `{"type": "tool", "name": "record_invoice"}`
- D) Leave it unset and ask for JSON in the prompt

<details markdown="1"><summary>Answer</summary>
**C**: forcing the one tool guarantees structured output with that schema. `any` is for when there are several schemas. → [Ch 9](../part-c/09-tool-design.html), [Ch 20](../part-e/20-structured-extraction.html)
</details>

**Q15.** An agent with 18 tools often picks the wrong one. What is the best fix?
- A) Scope the tools by role (about 4–5 per agent), or use tool search for large catalogs
- B) A longer system prompt
- C) A higher `max_tokens`
- D) `tool_choice: any`

<details markdown="1"><summary>Answer</summary>
**A**: fewer, relevant tools mean better choices. → [Ch 9](../part-c/09-tool-design.html)
</details>

**Q16.** The team wants everyone to share a GitHub MCP server without committing tokens. What should they do?
- A) Each developer adds it in user scope by hand
- B) Commit the token in `.mcp.json`
- C) Put it in `CLAUDE.md`
- D) Define it in the project `.mcp.json` with `${GITHUB_TOKEN}`

<details markdown="1"><summary>Answer</summary>
**D**: project scope gives everyone the same server, and env-var expansion keeps secrets out of git. → [Ch 12](../part-c/12-mcp-production.html)
</details>

**Q17.** The agent makes about 20 exploratory calls to learn a database layout before every task. Which MCP primitive fixes this?
- A) A tool
- B) A prompt
- C) A resource exposing the schema
- D) Sampling

<details markdown="1"><summary>Answer</summary>
**C**: resources provide read-only reference context up front. → [Ch 11](../part-c/11-mcp-fundamentals.html)
</details>

**Q18.** An MCP server needs an LLM to summarize logs but must not hold its own API key. What should it use?
- A) Sampling through the client
- B) Roots
- C) Elicitation
- D) Progress notifications

<details markdown="1"><summary>Answer</summary>
**A**: sampling lets the server borrow the **host's** model, with the host in control. → [Ch 12](../part-c/12-mcp-production.html)
</details>

---

## D3 · Claude Code (Q19–26)

**Q19.** A new team member's Claude Code doesn't follow the team's API conventions. The conventions are in the tech lead's `~/.claude/CLAUDE.md`. What is the fix?
- A) Ask the new member to copy the file
- B) Put them in `CLAUDE.local.md`
- C) Turn them into a skill
- D) Move them into the project `CLAUDE.md` (or `.claude/rules`) and commit it

<details markdown="1"><summary>Answer</summary>
**D**: the user level is personal. Team standards belong at project level, in git. → [Ch 14](../part-d/14-claude-md-rules.html)
</details>

**Q20.** Testing conventions apply to every `*.test.tsx` file, and those files are spread across 30 directories. Where should the conventions go?
- A) A CLAUDE.md in each directory
- B) `.claude/rules/testing.md` with a `paths:` glob
- C) The user-level CLAUDE.md
- D) A PostToolUse hook

<details markdown="1"><summary>Answer</summary>
**B**: path-scoped rules suit conventions that span many directories. → [Ch 14](../part-d/14-claude-md-rules.html)
</details>

**Q21.** You need to split a monolith into services, touching about 45 files, with several valid designs. How should you start?
- A) Direct execution
- B) `bypassPermissions` to go faster
- C) Plan mode: investigate, propose, approve, then execute
- D) Run it in CI with `-p`

<details markdown="1"><summary>Answer</summary>
**C**: large, ambiguous, multi-file work calls for plan mode first. → [Ch 13](../part-d/13-claude-code-architecture.html)
</details>

**Q22.** Which built-in tool finds every caller of `process_refund`?
- A) Glob
- B) Grep
- C) Read every file
- D) WebSearch

<details markdown="1"><summary>Answer</summary>
**B**: Grep searches **contents**. Glob matches file names. → [Ch 13](../part-d/13-claude-code-architecture.html)
</details>

**Q23.** The CI job that runs Claude Code hangs until it times out. What is the most likely fix?
- A) Use `-p` (print / non-interactive mode)
- B) Add `--verbose`
- C) Use `acceptEdits` mode
- D) Run `/compact` first

<details markdown="1"><summary>Answer</summary>
**A**: without `-p`, Claude Code waits for interactive input. → [Ch 17](../part-d/17-ci-cd-review.html)
</details>

**Q24.** Every edited Python file must **always** be formatted with ruff. What is the best mechanism?
- A) An instruction in CLAUDE.md
- B) A skill
- C) A PostToolUse hook on Edit and Write
- D) A formatter subagent

<details markdown="1"><summary>Answer</summary>
**C**: "always" means a deterministic **hook**. → [Ch 15](../part-d/15-skills-subagents-hooks.html)
</details>

**Q25.** The Claude session that wrote a feature then reviews it and misses obvious bugs. What is the best fix?
- A) Review with an independent instance that doesn't share the author's context
- B) Allow more turns
- C) Tell the same session to "be more critical"
- D) Raise the effort in the same session

<details markdown="1"><summary>Answer</summary>
**A**: the author session shares the same reasoning and blind spots. → [Ch 17](../part-d/17-ci-cd-review.html)
</details>

**Q26.** A team skill produces very verbose logs that fill the main conversation. Which frontmatter setting helps?
- A) `allowed-tools`
- B) `disable-model-invocation: true`
- C) `argument-hint`
- D) `context: fork`

<details markdown="1"><summary>Answer</summary>
**D**: `context: fork` runs the skill in an isolated subagent context. → [Ch 15](../part-d/15-skills-subagents-hooks.html)
</details>

---

## D4 · Prompts & Structured Output (Q27–34)

**Q27.** 40% of the CI reviewer's comments are about comment *wording*, and developers have started ignoring all its comments. What is the best fix?
- A) Add "be conservative" to the prompt
- B) Explicit criteria: flag only comments that contradict the code's behaviour, and don't flag style or wording
- C) Report only findings with confidence > 90%
- D) Add more examples of flagged comments

<details markdown="1"><summary>Answer</summary>
**B**: categorical flag / don't-flag criteria are what cut false positives. → [Ch 18](../part-e/18-prompt-engineering.html)
</details>

**Q28.** The same kind of bug gets "blocker" in one PR and "minor" in another. What is the fix?
- A) Ask the model to "be consistent"
- B) Set temperature to 0
- C) Define severity levels, each with a concrete code example
- D) Let the model rate its own severity confidence

<details markdown="1"><summary>Answer</summary>
**C**: a rubric with examples anchors the judgment. → [Ch 18](../part-e/18-prompt-engineering.html)
</details>

**Q29.** In extraction, about 4% of invoices get a made-up `due_date` that isn't in the document. What is the best fix?
- A) Make `due_date` nullable, with an example that returns `null`
- B) Make `due_date` required
- C) Retry until a date is found
- D) Raise the temperature

<details markdown="1"><summary>Answer</summary>
**A**: required non-null fields push the model to invent values. → [Ch 20](../part-e/20-structured-extraction.html)
</details>

**Q30.** Validation keeps failing for `tax_id`, and you confirm that the document doesn't contain one. What should you do?
- A) Increase retries to 10
- B) Use a bigger model
- C) Retry with the exact validation error
- D) Stop retrying: return `null` or flag the document for review

<details markdown="1"><summary>Answer</summary>
**D**: a retry can't create information that isn't in the source. → [Ch 20](../part-e/20-structured-extraction.html)
</details>

**Q31.** What should a validation retry request include?
- A) "Please try again"
- B) The original document, the failed output and the exact validation error
- C) Only the error message
- D) A new conversation without the document

<details markdown="1"><summary>Answer</summary>
**B**: Claude needs the source, its mistake and the precise problem. → [Ch 20](../part-e/20-structured-extraction.html)
</details>

**Q32.** An extraction passes schema validation, but the line items don't add up to the stated total. What should happen?
- A) Nothing. Schema-valid means correct
- B) Retry until they match
- C) Drop the line items
- D) Compute `calculated_total` in code and send mismatches to review

<details markdown="1"><summary>Answer</summary>
**D**: schemas catch syntax, not meaning. Expose the discrepancy rather than hide it. → [Ch 20](../part-e/20-structured-extraction.html)
</details>

**Q33.** 10,000 documents need processing overnight, cost matters, and results are needed by morning. What should you use?
- A) The Message Batches API, with a `custom_id` per document
- B) Synchronous parallel calls
- C) Streaming responses
- D) A multi-agent coordinator

<details markdown="1"><summary>Answer</summary>
**A**: non-blocking bulk work at about 50% of the cost. → [Ch 21](../part-e/21-batch-processing.html)
</details>

**Q34.** Which statement about the Message Batches API is TRUE?
- A) It is ideal for blocking pre-merge checks
- B) It can take up to 24h, so it is unsuitable for blocking workflows
- C) It supports multi-turn tool loops within one request
- D) Results come back in submission order

<details markdown="1"><summary>Answer</summary>
**B**: no latency guarantee, no tool loop inside a request, and results must be matched by `custom_id`. → [Ch 21](../part-e/21-batch-processing.html)
</details>

---

## D5 · Context & Reliability (Q35–40)

**Q35.** After 40 turns the support agent forgets the $740 refund it agreed to earlier and offers $500. What is the best fix?
- A) A bigger context window
- B) Summarize the history more aggressively
- C) A case facts block (IDs, amounts, promises), held in code and included in every request
- D) Restart the conversation

<details markdown="1"><summary>Answer</summary>
**C**: summaries lose exact values, so keep them in a structured block. → [Ch 22](../part-f/22-context-management.html)
</details>

**Q36.** A key finding buried in the middle of a long input keeps getting missed. What should you do?
- A) Put the key findings at the start, use clear section headers, and restate the critical points at the end
- B) Lower the temperature
- C) Increase `max_tokens`
- D) Remove the section headers

<details markdown="1"><summary>Answer</summary>
**A**: this is lost-in-the-middle. The start and end are used most reliably. → [Ch 22](../part-f/22-context-management.html)
</details>

**Q37.** Each `lookup_order` call returns 40 fields, and the context fills up after a few calls. What is the best fix?
- A) Prompt caching
- B) The Batch API
- C) A larger model
- D) Trim the tool results to the relevant fields

<details markdown="1"><summary>Answer</summary>
**D**: trim before the results enter the history. Caching doesn't reduce the context size. → [Ch 10](../part-c/10-tool-errors-results.html), [Ch 22](../part-f/22-context-management.html)
</details>

**Q38.** Two credible sources report different market-size figures. What should the report do?
- A) Use the more recent figure
- B) Average the two
- C) Report both, with attribution, marked as contested
- D) Leave both out

<details markdown="1"><summary>Answer</summary>
**C**: the disagreement is itself a finding. Don't hide it. → [Ch 23](../part-f/23-provenance-sources.html)
</details>

**Q39.** The final research report no longer shows which source supports each claim. What is the root fix?
- A) Ask the synthesizer to add a bibliography at the end
- B) Require the subagents to output structured claim–source mappings, and keep them through synthesis
- C) Use a bigger model
- D) Use fewer subagents

<details markdown="1"><summary>Answer</summary>
**B**: attribution is lost **upstream**, during compression. Keep it structured from the start. → [Ch 23](../part-f/23-provenance-sources.html)
</details>

**Q40.** Extraction accuracy is 97% overall. Should human review be dropped?
- A) Yes. 97% is high
- B) Yes, but only for invoices
- C) No. Remove the automation instead
- D) Not yet: check accuracy per document type and sample the high-confidence results first

<details markdown="1"><summary>Answer</summary>
**D**: an aggregate figure can hide weak segments, so verify them before reducing oversight. → [Ch 19](../part-e/19-prompt-evaluation.html)
</details>

---

## 🔑 Answer Key

| Q | Ans | Q | Ans | Q | Ans | Q | Ans |
|:--|:--|:--|:--|:--|:--|:--|:--|
| 1 | B | 11 | D | 21 | C | 31 | B |
| 2 | C | 12 | A | 22 | B | 32 | D |
| 3 | D | 13 | C | 23 | A | 33 | A |
| 4 | B | 14 | C | 24 | C | 34 | B |
| 5 | A | 15 | A | 25 | A | 35 | C |
| 6 | C | 16 | D | 26 | D | 36 | A |
| 7 | D | 17 | C | 27 | B | 37 | D |
| 8 | B | 18 | A | 28 | C | 38 | C |
| 9 | C | 19 | D | 29 | A | 39 | B |
| 10 | A | 20 | B | 30 | D | 40 | D |

**Scoring:** 36+ = ready · 30–35 = revise the chapters behind your wrong answers · < 30 = re-read the Exam Traps tables in Parts B–F.
