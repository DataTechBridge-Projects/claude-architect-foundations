---
title: Full Sample Exam (60 Questions)
parent: Practice Exam
nav_order: 1
---

# Full Sample Exam (60 Questions)

This is a full-length, timed practice exam spanning all five CCA-F domains: Agentic Architecture, Tool Design, Workflow Design, Prompt Design, and Context Management, plus one cross-layer synthesis question. Give yourself about 90 seconds per question (roughly 90 minutes total) and answer all 60 before looking at the Answer Key — the questions are grouped by domain so you can also use this as a focused drill on any one layer later.

## Agentic Architecture (Questions 1–13)

**1.** A dashboard pipeline pulls data from an API, feeds it into Claude for one summarization call, then renders a chart from the model's fixed-format response. No output ever influences what happens next. Is this pipeline agentic?
A. Yes, because it uses a large language model to make decisions.
B. Yes, because it produces natural-language output that a human reads.
C. No, because the model's output does not shape any subsequent action or observation — it's a single fixed call within a pipeline.
D. No, because Claude only becomes agentic when given more than one tool.

**2.** A support team wants to automate first-line ticket handling. The right next question genuinely depends on how the customer answers the previous one, and the number of possible paths is too large to enumerate as a fixed script. Which architecture best fits?
A. A fixed, hardcoded decision-tree pipeline, because it is easier to audit.
B. A single agent with autonomy to decide its own next action at each step, since the correct path can't be enumerated in advance.
C. A multi-agent system with one subagent per possible customer question.
D. No automation — this task must remain fully manual.

**3.** Which of the following is NOT, by itself, a sound justification for splitting a task across multiple agents rather than one?
A. Different stages of the task require genuinely distinct specialized expertise.
B. One stage produces so much exploratory noise that isolating it protects the rest of the system's context.
C. Two subtasks are fully independent and can be executed in parallel to save wall-clock time.
D. The task looks impressive to stakeholders if it involves several coordinating agents.

**4.** A finance team's month-end close process has eleven well-documented, always-in-the-same-order steps, each with a clearly defined input and output. Which architecture is the best fit?
A. A fixed workflow/pipeline, since the sequence is fully predictable and doesn't need runtime judgment to determine order.
B. A single autonomous agent given broad discretion over ordering.
C. A multi-agent system with a coordinator negotiating step order at runtime.
D. A fully manual process, since automation cannot apply to finance workflows.

**5.** When an orchestrator dispatches a subagent, what does the orchestrator actually observe from that subagent's work?
A. The subagent's full internal reasoning trace and every intermediate tool call it made.
B. Only what the subagent explicitly reports back, structurally identical to any other tool result.
C. Nothing; subagent results are not returned to the orchestrator at all.
D. The subagent's entire context window is merged into the orchestrator's.

**6.** In most systems, subagents start with fresh, isolated context. Under what circumstance does giving a subagent shared context with the orchestrator make sense?
A. Never — shared context always defeats the purpose of using a subagent.
B. Only when the subagent needs to call more than one tool.
C. When continuity with the orchestrator's exact reasoning and conversational state matters more than isolating the subagent from unrelated noise.
D. Only when the subagent is expected to fail.

**7.** A workflow step needs to generate a database migration script based on the output of an earlier step that inspects the current schema. A second step then needs to run that migration before a third step can verify it. How should these three steps be coordinated?
A. Sequentially, because each step has a genuine data dependency on the one before it.
B. Fully in parallel, to minimize wall-clock time.
C. It doesn't matter, since coordination pattern has no effect on correctness.
D. As three fully independent multi-agent teams.

**8.** A workflow decomposes "migrate the codebase to a new framework" into a single step: "do the migration." What is the main risk of this decomposition?
A. It has too many handoffs, adding coordination overhead.
B. It cannot be executed by an agent at all.
C. It requires a multi-agent system by definition.
D. It pushes all the implicit sub-decomposition burden onto whatever executes that one oversized step, with no intermediate checkpoint to catch problems early.

**9.** An agent is about to permanently revoke a customer's account access with no recovery path once executed. What is the appropriate safeguard?
A. A checkpoint that simply logs the action for later review.
B. No safeguard is needed if the agent has performed similar actions successfully before.
C. A human approval gate before the action executes, since the action is irreversible and high-blast-radius.
D. An automatic retry policy in case the first attempt fails.

**10.** A billing agent accidentally overcharges a customer's card. A refund can be issued to fully offset the charge. How should this action be classified?
A. Reversible, since the original charge can be literally undone.
B. Compensable, since a defined follow-up action (the refund) can counteract the effect, even though the original charge itself can't be un-made.
C. Irreversible, since money changed hands.
D. Low-risk, since payment errors are common and expected.

**11.** A user notices that a single tool call inside an otherwise-fine multi-step task returned bad data, but the agent kept reasoning from that bad result for the rest of the task. Which timeline did the actual failure occur on, and what does that imply about the fix?
A. The loop-iteration level, where one call produced bad output; the fix belongs there (e.g., validating that result) rather than at the task or session level.
B. The session level; the fix is to end the conversation and start a new one.
C. The subagent-lifespan level, even though no subagent was involved.
D. There is no meaningful distinction between these timelines.

**12.** As a tool set grows from three tools to thirty, why does structured tool use scale better than having the model describe its intended action in prose for a wrapper to parse?
A. Prose becomes cheaper in tokens as the tool set grows.
B. Structured tool use caps the number of tools a system can define.
C. Prose-based systems automatically deduplicate similar actions.
D. Each structured tool has its own name and schema, so new tools can be added independently without needing new prose-parsing logic to disambiguate free-text intent across all thirty.

**13.** A company is deciding how much autonomy to give an agent that drafts and sends marketing emails to customer segments. The task is repetitive and its steps are well understood, but a wrong send could damage the brand at scale. What is the most defensible design?
A. Full autonomy, since the task is repetitive and well understood.
B. No automation at all, since any mistake would be too costly.
C. A workflow (not full agentic autonomy) for the repetitive, well-understood parts, with the actual "send" action gated behind human approval given its high blast radius.
D. A multi-agent system where five agents vote on whether to send.

## Tool Design (Questions 14–24)

**14.** Why is tool design better understood as designing an API for a reasoning model rather than for a human developer?
A. Models cannot parse JSON schemas, so descriptions must be in plain English only.
B. The model cannot step outside the conversation to ask a clarifying question the way a human reading documentation could, so every ambiguity becomes a measurable rate of misuse.
C. Human developers never misread API documentation, while models always do.
D. Tool design for a model requires no description field at all.

**15.** A team is considering splitting one overloaded `manage_order` tool (with a `mode` parameter for create/update/cancel/refund) into four separate tools. Which of the following is NOT a legitimate reason to do so?
A. Each mode has a different, only-partially-overlapping set of required fields, and the combined schema is hard to specify unambiguously.
B. The four operations have meaningfully different risk profiles that should be permissioned separately.
C. Splitting will automatically make the total token cost of the tool set lower no matter how many tools result.
D. Separate tools let each operation's error responses be tailored to that specific failure mode.

**16.** A `file_bug_report` tool needs a field for "severity." Which design best follows sound guidance on structure versus prose in schemas?
A. A constrained enum (`low`, `medium`, `high`, `critical`), since severity is a small, well-defined set of options where structure aids validation and consistency.
B. A free-text string field, since severity is subjective and hard to define.
C. No field at all; let the model infer severity from the bug description at read time.
D. A numeric field with no defined range or meaning.

**17.** A `submit_expense` tool fails and returns only `Error: 500`. What is the main problem with this error response?
A. It is too long for the model to parse.
B. HTTP status codes are never valid in tool responses.
C. It reveals too much internal implementation detail.
D. It gives the model no actionable signal — no indication of what went wrong, whether it's retryable, or what a valid alternative would look like.

**18.** A `create_shipment` tool call fails because the destination postal code doesn't exist in the carrier's system. Should the harness retry the exact same call?
A. Yes, this is a transient failure that will likely succeed on a second attempt.
B. No — the failure is structural, not transient; retrying with the same invalid postal code can never succeed, so a corrected input or escalation is needed instead.
C. Yes, but only after waiting sixty seconds.
D. It doesn't matter, since retries are always free.

**19.** A tool call fails with a clear, well-known transient network timeout. Where should the retry logic for this specific case live?
A. The harness can handle it with a transparent, automatic retry, reserving the model's reasoning turns for genuinely ambiguous recovery decisions.
B. The model should reason through it in a full turn each time, since only the model can decide to retry.
C. It should always be escalated to a human immediately.
D. This kind of failure cannot be recovered from at all.

**20.** What best describes the division of responsibility between an MCP server and an MCP client?
A. The server discovers and connects to clients; the client exposes tools for a specific external system.
B. There is no meaningful distinction; the terms are interchangeable.
C. The server knows about one external system and exposes its tools, resources, and prompts through the protocol; the client (e.g., built into an application) discovers and connects to servers, making those tools available to the model.
D. The client is responsible for authenticating the model itself, while the server renders output to the user.

**21.** An organization connects a newly built internal MCP server with a single shared admin-level credential used by every team that connects to it. What governance principle does this violate?
A. Schema validity, since the server's tool schemas are presumably malformed.
B. Structured tool use, since MCP tools are not schema-validated.
C. Granularity matching, since the server likely has too many tools.
D. Least-privilege credential scoping — every connecting team inherits far more access than most of their tasks require.

**22.** A `run_query` tool has a minimal schema (just a `sql` string field) but can execute arbitrary SQL against a production database, including destructive statements. What should actually provide safety here?
A. The JSON Schema itself, by restricting the string field's format.
B. Permissions, scoping (e.g., a read-only credential or query allowlist), and sandboxing around the tool — not the schema, which only validates that a string was provided.
C. Nothing; broad tools like this should never be exposed regardless of surrounding controls.
D. Renaming the tool to something less alarming.

**23.** A coding-assistant tool set offers one broad `Bash` tool for arbitrary shell commands but three separate, narrow tools for file search. Why is this inconsistency in granularity defensible?
A. Granularity should match each capability's own shape — shell commands span an effectively unbounded space best left broad and governed by permissions, while search operations are few, distinct, and well served by narrow, purpose-built tools.
B. It isn't defensible; all tools should use the same granularity for consistency.
C. Broad tools are always technically superior and narrow tools are only used for legacy reasons.
D. The inconsistency exists purely by historical accident with no design rationale.

**24.** A model is repeatedly given two tools, `update_ticket` and `close_ticket`, and often calls `update_ticket` with a status change when it should call `close_ticket` directly. What is the most productive first diagnosis?
A. The model is fundamentally incapable of selecting between two similar tools.
B. The tools must be merged into a single tool immediately, with no other option considered.
C. The tool descriptions likely don't clearly state when `close_ticket` should be preferred over a generic status update via `update_ticket` — selection is relative to the full set of alternatives.
D. This is purely a context-window sizing problem unrelated to tool definitions.

## Workflow Design (Questions 25–33)

**25.** A company's engineering org wants Claude to know that this specific repository's API layer must never be called directly from UI components — always through the service layer. Where does this constraint belong?
A. In the harness-level system prompt, since it should apply company-wide.
B. In a Skill that must be manually invoked before every UI change.
C. Nowhere; this should just be explained fresh in each prompt.
D. In this repository's `CLAUDE.md`, since it's a project-specific convention that should be automatically available whenever anyone works in this repo.

**26.** A legal team's Claude-based assistant needs access to the company's entire, constantly-updated contract repository — tens of thousands of documents — but any single query only ever needs a handful of them. What's the best design?
A. Paste the entire contract repository into `CLAUDE.md` so it's always available.
B. Retrieval: index the repository and pull in only the specific documents relevant to each query.
C. A system prompt update every time a contract changes.
D. A Skill containing the full text of every contract.

**27.** A team has a well-defined but only occasionally-needed procedure for generating a compliance audit report, involving several specific steps and a couple of bundled scripts. Should this be a Skill or a `CLAUDE.md` entry?
A. A Skill, since it's a procedural, occasionally-needed capability better loaded on demand than paid for on every turn.
B. A `CLAUDE.md` entry, so it's loaded automatically on every session regardless of task.
C. Neither; this must always be a separate subagent.
D. A system prompt addition.

**28.** A task needs its own isolated context window and its own independent model call, reporting a result back without polluting the calling agent's ongoing context. Which mechanism actually provides that?
A. A Skill, since Skills always run in isolated sandboxes.
B. Either mechanism works identically for this purpose.
C. A subagent, since a Skill loads into the current agent's existing context and thread rather than spinning up a separate one.
D. A `CLAUDE.md` entry configured with an isolation flag.

**29.** A workflow step converts a set of currency amounts using a fixed, provided exchange-rate table. Should this step be deterministic or agentic?
A. Agentic, because it involves numbers and could theoretically be tricky.
B. It must be handled by a multi-agent system for accuracy.
C. It should always be done by a human.
D. Deterministic, since the correct output is fully specified by the input and the rate table, with no judgment required.

**30.** A research task requires the model to decide, at each point, whether the evidence gathered so far is sufficient or whether another search is needed — the number of searches can't be fixed in advance. What does this argue for?
A. A fixed pipeline with a hardcoded number of search steps.
B. Agentic control over the sequencing itself, since the right next step depends on what was just discovered.
C. Converting the entire task into a `CLAUDE.md` entry.
D. Skipping search entirely and relying on the model's training data.

**31.** An architect is asked whether every team's `CLAUDE.md` must follow an identical section-by-section template company-wide. What's the best answer?
A. Not necessarily — many workflow conventions like file structure and formatting are legitimately local to a team or repo; only genuinely cross-cutting concerns like security-relevant hard constraints usually warrant a centralized standard.
B. Yes, absolute uniformity is always required for maintainability.
C. No, `CLAUDE.md` files should never share any convention across teams.
D. This decision should be made exclusively by whichever team shipped first.

**32.** A developer manually pastes code diffs into a chat interface for review before merging, whenever they remember to. What does this workflow lack to be considered properly CI/CD-integrated?
A. Nothing; manual invocation is itself a valid form of CI/CD integration.
B. A larger context window for the review model.
C. A defined trigger tied to the pipeline, reproducible input, durable captured output, and explicit gating — the review currently has none of these consistently.
D. Conversion of the review into a subagent.

**33.** Which of the following is NOT a property of a well-placed workflow validation or review gate?
A. It's placed at a point where the cost of an undetected error would be high.
B. It's applied uniformly at every single step regardless of stakes, to be maximally safe.
C. It can combine deterministic validation with human review depending on the stakes involved.
D. It's integrated into the surrounding process rather than left as an unstated assumption.

## Prompt Design (Questions 34–46)

**34.** A prompt instructs Claude to "write a product description." Output length and tone vary wildly across runs. What's the most likely fix?
A. Add explicit success criteria: target length, audience, tone, and what to prioritize when trade-offs arise.
B. Increase the temperature setting.
C. Add ten few-shot examples of unrelated product descriptions.
D. Switch to a smaller model to reduce variance.

**35.** A customer-facing chatbot inserts the raw text of an inbound customer email directly into the system prompt, alongside the bot's core behavioral rules. What risk does this create?
A. None; system prompts are the correct place for all content, trusted or not.
B. It will cause the API to reject the request outright.
C. Because system-level content is trusted more heavily than user content, blending in untrusted customer text there raises the risk of that text being treated as an instruction (prompt injection).
D. It reduces token cost compared to placing the email in user content.

**36.** A support team wants Claude to always respond in a very particular clipped, informal tone that's difficult to fully capture in a written style guide. What technique is best suited here?
A. A longer and more detailed prose description of the tone.
B. Few-shot examples that demonstrate the tone directly, since it's easier to show than to fully specify in prose.
C. Lowering the temperature to zero.
D. Removing all instructions and letting the model infer tone from the user's message alone.

**37.** A team building a spam-classification prompt selects ten few-shot examples, all of which are typical, easy-to-classify spam emails. What's the risk?
A. The prompt will exceed the token budget.
B. There is no risk; more typical examples always improve reliability.
C. The model will refuse to classify any email that doesn't closely match one of the ten examples.
D. The examples don't cover the range of real production inputs (including ambiguous or edge-case emails), so the model may perform poorly exactly where classification is hardest.

**38.** A few-shot prompt for classifying support tickets as "urgent" uses five examples, all of which are written in all-caps. A new ticket written in all-caps but about a trivial issue gets misclassified as urgent. What likely happened?
A. The model plausibly learned "written in all-caps" as an unintended proxy for urgency, since that incidental detail was shared across every example.
B. The model correctly identified urgency based on tone.
C. This is unrelated to the few-shot examples and is a pure model error.
D. All-caps text always indicates urgency, so this is correct behavior.

**39.** A production pipeline parses every one of Claude's responses as JSON downstream. The current prompt only asks politely for "valid JSON only." What's the more reliable fix?
A. Ask even more politely, with additional emphasis in all caps.
B. Increase the temperature to encourage more creative formatting.
C. Enforce structure at the mechanism level — schema-constrained generation or a tool-call-as-output pattern — rather than relying on wording alone.
D. Remove the JSON requirement, since Claude cannot reliably produce structured output.

**40.** An internal step's output is consumed only by the next automated step in a pipeline, never read directly by a person. How should that output be optimized?
A. For warmth and conversational tone, since all output should read naturally.
B. For terse, unambiguous parseability, since optimizing for a human reader that doesn't exist wastes effort and risks introducing parsing friction.
C. For maximum length, to capture as much nuance as possible.
D. It doesn't matter, since format has no effect on downstream automated consumers.

**41.** A single prompt asks Claude to draft a legal clause and then, in the same response, critique and revise its own draft. What's the main limitation compared to a separate reviewer pass?
A. It always costs less than a separate pass, which is itself a limitation.
B. Claude is architecturally incapable of revising its own output.
C. It cannot be used together with any structured output technique.
D. The critique shares the same context, assumptions, and reasoning momentum as the original draft, so it's poorly positioned to catch errors rooted in those same assumptions.

**42.** A team runs a low-stakes internal tool that generates suggested Slack-message drafts a human always reads before sending. They're debating adding a full independent-reviewer pass before every draft is shown. What's the best judgment call?
A. Given the low stakes and the human-in-the-loop read before sending, a full independent review pass is likely not worth its added latency and cost.
B. Always add the heaviest available review, since more verification is strictly better.
C. Skip all quality control, since a human eventually reads it anyway.
D. Independent review is legally required for any AI-generated text.

**43.** What distinguishes a prompt treated as an engineered system component from a prompt treated as a one-off first draft?
A. Engineered prompts are always longer than draft prompts.
B. Engineered prompts never contain examples.
C. Engineered prompts have tested coverage across a realistic range of inputs, tracked changes over time, and defined behavior on failure — not just success on the one case the author had in mind.
D. Engineered prompts are written exclusively in the system role.

**44.** Which of the following is NOT a good reason to add few-shot examples to an already-clear, unambiguous instruction?
A. Reinforcing a hard-to-describe stylistic pattern.
B. Making the prompt feel more thorough, even though the instruction alone already produces reliable, correct output.
C. Demonstrating a fuzzy classification boundary.
D. Showing an idiosyncratic formatting convention that's easier to demonstrate than describe.

**45.** A team runs the same extraction prompt across ten thousand structured records overnight and needs consistent, repeatable field extraction across the whole batch. What temperature setting best fits this goal?
A. A high temperature, to maximize creative variation across records.
B. Temperature has no effect on consistency, only on latency.
C. Temperature should be increased partway through the batch to avoid repetitive output.
D. A low temperature, since minimizing randomness in token selection best supports consistency across a large batch.

**46.** A team wants Claude's output to always conform to a strict structured schema and decides to define a `submit_result` function whose input schema is that structure, instructing the model to always call it — even though the function performs no real external action. What pattern is this?
A. The tool-call-as-output pattern, which repurposes a validated tool schema as the structured-output contract.
B. Retrieval-augmented generation.
C. Few-shot prompting.
D. Context compaction.

## Context Management (Questions 47–59)

**47.** A long-running agent session rewrites its last twenty turns of exploratory back-and-forth into a three-paragraph narrative describing what was tried, what was learned, and why the current approach was chosen. Which technique is this?
A. Discarding.
B. Compaction.
C. Summarization, since it's an editorial rewrite that preserves decisions and their justification rather than a mechanical trim.
D. Retrieval.

**48.** A coding agent compresses its context automatically the instant token usage crosses 80% of the window, even if that happens in the middle of writing a single function. What's the main risk of this specific trigger design?
A. Compression may land mid-thought, at an arbitrary point rather than a natural seam in the work, unlike a milestone-based trigger tied to completed units of work.
B. It will never actually trigger, since 80% is too high a threshold.
C. Threshold-based compaction cannot be combined with summarization.
D. It has no risk; threshold-based triggers are always superior to milestone-based ones.

**49.** A subagent is dispatched to fix a bug, but the orchestrator withholds all prior conversation about why the bug matters to the customer, giving only the raw stack trace. The subagent fixes the literal error but breaks an unstated business requirement. What does this illustrate?
A. This proves subagents should always receive the orchestrator's full context.
B. This is purely a tool design failure unrelated to context.
C. This shows fresh-context subagents are never appropriate.
D. Over-isolation: withholding context the subagent genuinely needed produced a worse outcome, showing isolation should be scoped to task needs, not applied blanket.

**50.** An agent processing a batch of 200 reimbursement requests finds that 4 have a receipt image too blurry to read the amount. What's the best-granularity response?
A. Halt all 200 requests until every blurry receipt is manually resolved.
B. Process the 196 clear requests normally and escalate just the 4 blurry ones with the relevant receipt images attached.
C. Guess a plausible amount for each blurry receipt so the batch completes without interruption.
D. Silently drop the 4 blurry ones and report only 196 as the full batch.

**51.** An agent misreads a contract's effective date early in a multi-step review, and every subsequent step — deadline calculations, notice periods, renewal windows — builds on that wrong date, producing an internally consistent but entirely wrong final report. Why is this kind of error especially dangerous?
A. Because it involves a date, and dates are inherently harder to reason about than other data types.
B. Because this can only happen in multi-agent systems.
C. Because each downstream step looks individually reasonable given what came before, so the final output appears well-reasoned even though it rests on a bad premise — making the error easy to miss from the output alone.
D. Because the agent used too high a temperature setting.

**52.** A pricing agent is about to finalize a discount based on a customer tier it read early in a long session. Before committing, the system re-confirms that tier directly against the current billing system rather than trusting what's still sitting in context. What technique is this?
A. Re-verification — actively re-confirming a specific, load-bearing conclusion against ground truth before acting on it.
B. Compaction.
C. Summarization.
D. A milestone-based checkpoint, since it happens at a defined point in the task.

**53.** A three-file internal utility script, maintained by one person and rarely changed, is given an elaborate semantic-search retrieval pipeline to select relevant context on every query. What's wrong with this design?
A. Nothing; retrieval is always the superior choice regardless of project size.
B. Semantic search cannot technically be applied to small codebases.
C. This design is required for consistency across the organization.
D. It adds real engineering complexity and latency to solve a scale problem this tiny, stable project doesn't actually have — a lean, always-loaded static context would fit better.

**54.** A developer needs the model to understand one small, tightly cohesive configuration file where every line depends on the others to make sense. Should context selection pull the whole file or just the specific symbol being changed?
A. Just the specific symbol, since that always minimizes token cost.
B. The whole file, since its high cohesion means stripping it to one symbol risks losing context that clarifies how that symbol should be used.
C. Neither; configuration files should never be loaded into context.
D. Half the file, split arbitrarily by line count.

**55.** A human approval gate for high-value refunds shows the reviewer the entire 40-turn agent transcript instead of the specific refund amount, customer history, and stated reason. What's the likely consequence?
A. None; more information always makes for a better-informed reviewer.
B. It will cause the API to reject the approval request.
C. The same signal-dilution problem that affects models: the reviewer skims or takes far longer than the gate's value justifies, and the gate exists on paper without functioning well in practice.
D. It guarantees the reviewer approves every request faster.

**56.** In the same context, a live inventory system says an item is in stock, while a support ticket from three months ago says the item was permanently discontinued. Which is the best resolution approach?
A. Weigh source authority, currency, and verifiability together — a live system-of-record generally outranks a stale ticket here — and flag the conflict explicitly if it doesn't resolve clearly.
B. Always trust whichever source appears more recently in the conversation transcript.
C. Always defer to whichever source uses more confident, detailed language.
D. Split the difference and tell the customer the item is "possibly" in stock.

**57.** Which of the following is NOT a legitimate reason to isolate a subagent's context from the orchestrator's?
A. The subagent's exploration will generate a lot of noisy dead ends that shouldn't pollute the orchestrator's working set.
B. The subagent's task is narrow enough that it doesn't need the orchestrator's full history to do good work.
C. Keeping the subagent's context scoped makes its behavior more predictable and easier to evaluate.
D. Isolating context is required in every case purely because subagents exist, regardless of what the specific task needs.

**58.** A team restructures their prompt so that the system instructions, `CLAUDE.md` content, and tool definitions are placed first and never change between calls in a session, while live conversation and freshly retrieved documents are appended afterward. Why does this ordering matter?
A. It has no effect; the API processes all context identically regardless of order.
B. Stable, unchanging content placed early benefits from a reusable cached prefix, while volatile content downstream doesn't invalidate that cache on every turn.
C. It ensures tool definitions are hidden from the model until called.
D. It reduces the model's context window size.

**59.** A developer needs to find every call site of a specific function across a massive, actively developed monorepo, starting from one known usage. Which retrieval approach best fits this specific need?
A. Loading the entire monorepo into context at once.
B. A single static `CLAUDE.md` entry listing every call site by hand.
C. Structural/symbol-level navigation via the call graph, since the need is to follow a concrete code relationship rather than match loosely on meaning.
D. Pure semantic/embedding search over natural-language descriptions of the code.

## Cross-Layer Synthesis (Question 60)

**60.** A retail company's Claude-based agent autonomously approves or denies return requests. Recently, a batch of denials turned out to be wrong: a `check_inventory` tool had begun silently returning stale cached data during a vendor outage, but its error response gave no indication anything was wrong — it just returned old numbers in the normal success shape. The agent treated the stale numbers as current, denied several valid returns, and each denial cited the (wrong) inventory figures as justification, compounding the appearance of a well-reasoned decision. Which single change would have most directly prevented this failure from reaching customers?
A. Adding more subagents to the return-approval process for redundancy.
B. Redesigning the tool's response to signal degraded or stale data explicitly (a Tool-layer fix), which would have let the agent recognize it lacked trustworthy grounds to deny and escalate instead of confidently propagating a wrong inventory figure through its reasoning (a Context-layer failure mode).
C. Lowering the temperature setting on the model used for return decisions.
D. Adding more few-shot examples of correctly approved returns to the prompt.

## Answer Key

**1. C** — A single fixed call whose output never feeds back into a subsequent decision is not agentic, regardless of model size or output quality.

**2. B** — When the correct path can't be enumerated in advance, a single agent with autonomy over its own next step is the right fit, not a hardcoded tree or a one-subagent-per-path system.

**3. D** — Perceived sophistication is not a legitimate architectural justification; specialization, context isolation, and genuine parallelism are.

**4. A** — A fully predictable, always-ordered sequence is the textbook case for a fixed workflow rather than agentic or multi-agent control.

**5. B** — A subagent dispatch behaves like any other tool call: the orchestrator only sees the explicitly reported result, not the subagent's internal loop.

**6. C** — Shared context is the exception, justified specifically when continuity with the orchestrator's own reasoning and state outweighs the benefits of isolation.

**7. A** — Each step depends on the actual output of the step before it, which is the defining case for sequential rather than parallel coordination.

**8. D** — An oversized, undecomposed step pushes all implicit sub-decomposition onto its executor with no intermediate point to catch problems.

**9. C** — Irreversible, high-blast-radius actions need a human approval gate before execution, not a passive checkpoint or unconditional trust in past success.

**10. B** — A refund is a defined counteracting action, which is exactly what makes an effect compensable rather than reversible or irreversible.

**11. A** — The bad data originated and was consumed within a single loop iteration; the fix (validating that result) belongs at that level, not at the task or session level.

**12. D** — Distinct names and schemas let new tools be added independently, avoiding the need for shared prose-parsing logic that would otherwise have to disambiguate free text across the whole growing set.

**13. C** — Splitting the well-understood repetitive work into a workflow while gating the irreversible, high-stakes "send" action behind human approval matches autonomy to actual risk.

**14. B** — A model can't ask an out-of-band clarifying question, so every ambiguity in a tool definition becomes a real, measurable rate of misuse.

**15. C** — Splitting tools doesn't automatically reduce total token cost; that depends on usage patterns, not the mere existence of more tools.

**16. A** — A small, well-defined set of severity levels is exactly the case where a constrained enum aids validation and consistency over free text.

**17. D** — A bare status code carries none of the diagnostic detail — cause, retryability, or valid alternative — that a reasoning model needs to act well.

**18. B** — An invalid postal code is a structural failure; identical retries can never succeed, so correction or escalation is required instead.

**19. A** — Clearly transient, mechanical failures are well suited to silent harness-level retries, reserving model reasoning turns for genuinely ambiguous recovery.

**20. C** — A server exposes one external system's tools/resources/prompts through MCP's standard shape; a client discovers and connects to servers on behalf of the model.

**21. D** — A single shared, over-privileged credential violates least-privilege scoping, handing every connecting team more access than most of their tasks need.

**22. B** — The schema only validates that a string was supplied; actual safety for a broad, high-capability tool comes from permissions, scoping, and sandboxing around it.

**23. A** — Shell commands span an unbounded space best governed by surrounding permissions, while a handful of distinct search operations are well served by narrow tools — granularity tracks each capability's own shape.

**24. C** — Selection failures between similar tools are almost always definition failures: the descriptions didn't clearly carve out when to prefer one over the other.

**25. D** — A repository-specific convention that should apply automatically whenever anyone works in that repo is exactly what `CLAUDE.md` is for.

**26. B** — Large, dynamic, query-specific knowledge where only a small subset is ever relevant is the textbook case for retrieval rather than standing context.

**27. A** — An occasionally-needed, well-defined procedure is better packaged as a Skill loaded on demand than paid for in every session via `CLAUDE.md`.

**28. C** — Only a subagent spins up its own context window and makes independent model calls; a Skill loads into the current agent's existing thread.

**29. D** — A fixed rate table fully specifies the correct output for any input, with no judgment required — the definition of a deterministic step.

**30. B** — When the right next step depends on what was just discovered, the sequencing itself needs agentic control rather than a fixed step order.

**31. A** — Many workflow conventions are legitimately local to a team or repo; centralization is generally reserved for cross-cutting, security-relevant constraints.

**32. C** — Ad hoc, memory-dependent review lacks a defined trigger, reproducible input, durable output, and explicit gating — the pillars of real CI/CD integration.

**33. B** — Gates should be placed where error cost is high, not applied uniformly everywhere regardless of stakes; uniform application is the anti-pattern, not the goal.

**34. A** — Vague framing with no stated length, audience, or priorities is the most common cause of wildly inconsistent open-ended output; explicit success criteria fix it directly.

**35. C** — System-level content is trusted more heavily than user content, so blending in untrusted customer text there raises real prompt-injection risk.

**36. B** — Tone and voice are easier to demonstrate through examples than to fully pin down in prose description.

**37. D** — Ten near-duplicate, typical examples don't cover the edge cases and ambiguity that real production inputs actually contain.

**38. A** — A detail shared across every example (all-caps formatting) can be learned as an unintended proxy for the actual target concept.

**39. C** — Polite wording is a probabilistic nudge, not a mechanical guarantee; schema-constrained generation or tool-call-as-output enforces structure at the mechanism level.

**40. B** — Output consumed only by another system should optimize for terse, unambiguous parseability rather than a human tone that has no reader to serve.

**41. D** — An inline critique inherits the same context and assumptions as the draft it's reviewing, limiting what errors it can actually catch.

**42. A** — Low stakes plus an existing human read-before-send step make a full independent review pass an added cost unlikely to be worth its latency here.

**43. C** — Tested coverage across realistic inputs, tracked changes, and defined failure behavior are what separate an engineered prompt from a one-off draft.

**44. B** — Adding examples purely to seem thorough, when the instruction is already reliable on its own, adds cost without adding reliability.

**45. D** — Lower temperature reduces randomness in token selection, which is what supports consistent, repeatable extraction across a large batch.

**46. A** — This is the tool-call-as-output pattern: a tool's validated input schema is repurposed as the structured-output contract even though the "tool" performs no real action.

**47. C** — Rewriting verbose turns into a narrative that preserves decisions and their justification is the defining move of summarization, not mechanical compaction.

**48. A** — A pure token-threshold trigger can fire mid-task at an arbitrary point, unlike a milestone-based trigger tied to a natural seam in the work.

**49. D** — Withholding context the subagent genuinely needed caused a worse outcome — exactly the over-isolation failure mode, not proof that sharing everything is always right.

**50. B** — Completing the unambiguous majority while escalating only the specific ambiguous cases, with relevant context attached, is the correctly granular response.

**51. C** — Each downstream step looks individually reasonable given what came before, camouflaging the flawed premise and making the final output look well-reasoned despite being wrong.

**52. A** — Actively re-confirming a load-bearing conclusion against ground truth before acting on it is precisely what re-verification means.

**53. D** — A small, stable, single-purpose project doesn't have the scale problem retrieval solves; the added complexity and latency aren't justified here.

**54. B** — High file cohesion means stripping context down to one symbol risks losing the surrounding detail that clarifies correct usage.

**55. C** — An unfiltered, lengthy transcript causes the same signal-dilution problem in a human reviewer that it would in a model, undermining the gate's actual function.

**56. A** — Resolving conflicting sources requires weighing authority, currency, and verifiability together, with unresolved conflicts flagged rather than silently decided by recency or tone alone.

**57. D** — Isolation should be scoped to what a specific task actually needs, not applied as a blanket rule just because a subagent exists.

**58. B** — A stable, unchanging prefix benefits from a reusable cache, while volatile content placed downstream avoids invalidating that cache on every turn.

**59. C** — Following a concrete code relationship like "who calls this function" is exactly what structural/symbol-level call-graph navigation is built for, more so than loose semantic matching.

**60. B** — The root cause was a Tool-layer error response that failed to signal degraded data, which let a Context-layer failure (an unverified, stale fact treated as trustworthy) propagate into consequential decisions; fixing the tool's error signaling addresses the actual failure point, unlike the other options.

<!-- prevnext:start -->

---

| [&larr; Previous: Practice Exam](./) | [Next: Course Conclusion &rarr;](../11-course-conclusion/) |
|:---|---:|

<!-- prevnext:end -->
