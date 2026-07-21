---
title: "Quiz Prep: The Tool Layer (20 Questions)"
parent: The Tool Layer
nav_order: 15
---

# Quiz Prep: The Tool Layer (20 Questions)

Twenty practice questions covering the Tool Layer module: tool definition and selection, structure versus prose, error response design, error recovery, granularity, MCP architecture and governance, and Claude Code's built-in tools. Work through all twenty before checking the Answer Key at the end.

**1.** Why is tool design best understood as "API design for a model rather than a human caller"?
A. Because models cannot read JSON.
B. Because the model, unlike a human developer, cannot ask an out-of-band clarifying question — every ambiguity in the definition becomes a real rate of wrong calls.
C. Because models only respond to natural language, never structured data.
D. Because human callers never make mistakes when reading documentation.

**2.** A tool set has `search_issues`, `search_pull_requests`, and `search_all`, but a model frequently calls the wrong one for exploratory queries. What is the most likely root cause, per this module?
A. The model is not capable enough to use three tools at once.
B. The descriptions don't clearly carve up when to use each tool relative to the others.
C. JSON Schema cannot support more than two similar tools.
D. The tool names are too long.

**3.** Which of the following best describes the risk of a tool set with many narrow, overlapping tools?
A. It reduces token cost on every call.
B. It multiplies the pairwise disambiguation burden the model has to reason through correctly as the set grows.
C. It eliminates the need for descriptions entirely.
D. It always produces faster responses than one broad tool.

**4.** Which of the following best describes the risk of a single, overloaded tool with many optional parameters?
A. It is always slower to execute than several narrow tools.
B. It pushes the disambiguation burden into a tangled, cross-field parameter space that's harder to specify unambiguously than separate tools would be.
C. It cannot be validated by JSON Schema at all.
D. It requires the model to make multiple tool calls instead of one.

**5.** According to the module, when should a schema field be free-text prose rather than a structured, constrained field?
A. Whenever the engineer is short on time.
B. Only when the content genuinely requires judgment or open-ended variation that a fixed schema can't anticipate.
C. Always, since prose is more flexible than structure in every case.
D. Never — prose fields should be eliminated from all tool schemas.

**6.** A tool set includes only a `get_user_by_id` lookup tool. A model receives a stale or mistyped ID and has no way to proceed. What principle does this failure illustrate the absence of?
A. Error response design.
B. Multiple pathways — no fallback route (such as a search-by-name tool) exists for this failure.
C. MCP governance.
D. Granularity matching for shell tools.

**7.** What is the single most load-bearing piece of information a tool's error response can convey, per the error-response-design lecture?
A. The HTTP status code of the underlying request.
B. Whether the failure is retryable or not, since the correct next action differs completely between the two.
C. The exact internal stack trace of the failure.
D. The total number of times the tool has ever failed historically.

**8.** Why does the module recommend against returning a raw stack trace as a tool's error result?
A. Stack traces are always longer than 100 words.
B. It's optimized for a human debugging source code, not a reasoning model deciding what to do next, and it can leak internal implementation detail.
C. Stack traces are not valid JSON.
D. Models are incapable of parsing any text containing file paths.

**9.** A `create_calendar_event` tool fails due to a scheduling conflict. Which error response best follows this module's guidance?
A. `Error: 409 Conflict`
B. `ConflictError at line 482 in calendar_service.py`
C. "Cannot create event: the slot conflicts with an existing event, 'Budget Review' (1:30pm-3pm). Not retryable with the same time — choose a different slot or use list_events to see what's scheduled."
D. "The operation could not be completed."

**10.** Per the module, what is the harness's typical role versus the model's typical role in error recovery?
A. The harness should never intervene; all recovery must be reasoned through by the model in-context.
B. The harness can transparently retry clearly transient failures and enforce budgets/escalation, while the model's reasoning is reserved for ambiguous cases like modified retries or choosing a fallback tool.
C. The model should always be given raw system logs so it can decide everything itself.
D. The harness and the model have identical, interchangeable responsibilities.

**11.** Which failure type should generally NOT be retried with unchanged arguments, per this module?
A. A momentary network timeout.
B. A rate-limit response suggesting a brief backoff.
C. A permission error where no available credential grants the required access.
D. A transient lock-contention error.

**12.** What is the recommended trigger for escalating a failure to a human rather than continuing to retry or fall back?
A. As soon as the very first tool call of the session fails.
B. When retries and fallback paths are genuinely exhausted, when the task's underlying assumptions appear wrong, or when continuing unsupervised is itself the risk.
C. Only after the harness has made exactly one attempt.
D. Escalation should never happen; agents should always resolve failures autonomously.

**13.** What does MCP (Model Context Protocol) primarily standardize?
A. The model's internal reasoning process.
B. A consistent way to expose tools, resources, and prompts from external systems to any compatible LLM client, rather than requiring bespoke integrations per system.
C. The pricing of API calls to Claude.
D. The wording of system prompts across applications.

**14.** In MCP's architecture, what is the primary responsibility of an MCP server?
A. To discover and connect to LLM clients.
B. To know about a specific external system and expose its tools, resources, and prompts through the protocol's standard shape.
C. To render the model's final text response to the end user.
D. To store the conversation history for the client application.

**15.** From the model's perspective, how does a tool exposed through an MCP server differ from a tool defined directly in application code?
A. It requires a fundamentally different calling convention.
B. It is not structured, unlike other tools.
C. It looks essentially the same — a name, description, and schema, invoked the same structured way — since MCP standardizes plumbing, not how the model perceives tools.
D. It cannot return errors.

**16.** An organization has dozens of MCP servers connected across teams, of varying provenance and quality. Which governance control does this module identify as essential?
A. Banning all MCP servers organization-wide.
B. A vetting process for which servers are trusted, least-privilege credential scoping per server, and audit logging of actions taken through them.
C. Allowing any team to connect any server without review, since MCP is inherently secure.
D. Relying solely on the model's judgment not to misuse a connected server's tools.

**17.** Why is Claude Code's `Bash` tool considered safe enough to expose despite its broad capability, per this module's analysis?
A. Its JSON Schema itself restricts which commands can be run.
B. Safety comes from infrastructure around the tool — permission prompts, allow/deny lists, and sandboxing — not from the schema, which is minimal.
C. It is not actually capable of modifying any files.
D. The model is contractually prohibited from calling it.

**18.** Why does Claude Code split file operations into `Read`, `Write`, and `Edit` rather than one `file_operation` tool with a mode parameter?
A. To increase the total token cost of every call.
B. Because reading, overwriting, and targeted editing have different risk profiles and different argument shapes, and separating them avoids a tangled, cross-field-dependent schema.
C. Because JSON Schema cannot express more than one operation type.
D. Because the underlying filesystem requires three separate permissions.

**19.** Why does Claude Code require a file to be read before it can be edited, in most configurations?
A. To reduce the number of available tools.
B. To prevent a blind edit against content the model hasn't verified, reducing the odds of a confidently wrong edit based on stale assumptions.
C. Because the `Edit` tool cannot function without a prior `Read` call's token count.
D. It is purely a legacy default with no design rationale.

**20.** What is the underlying lesson of Claude Code exposing both a single broad `Bash` tool and several narrow, well-differentiated tools like `Grep` and `Glob` in the same tool set?
A. Broad tools are always superior to narrow ones.
B. Narrow tools are always superior to broad ones.
C. Granularity should be matched per capability based on how distinct and how frequently-needed the underlying operations are, not applied as one global policy.
D. Tool granularity has no measurable effect on model behavior.

## Answer Key

**1. B** — The model can't seek out-of-band clarification the way a human developer reading docs can, so every ambiguity in a tool's definition becomes a real, measurable rate of misuse.

**2. B** — Selection failures among similar tools are almost always definition failures: the descriptions didn't carve up when to use each tool relative to its neighbors.

**3. B** — More narrow, overlapping tools means more pairwise boundaries the model must correctly reason about, and that burden grows faster than the raw tool count.

**4. B** — An overloaded tool pushes complexity into a tangled parameter space where cross-field constraints are hard to enforce or describe unambiguously.

**5. B** — Prose is the fallback for genuine judgment or variation; structure is the default because it's what makes a call validatable.

**6. B** — This is exactly the "multiple pathways" failure mode: no fallback route exists once the primary path (lookup by ID) fails.

**7. B** — Retryable versus non-retryable is the single most consequential signal, because it determines the correct — and opposite — next action.

**8. B** — Stack traces are built for a human debugging source code; they give a reasoning model little actionable guidance and can leak internal detail.

**9. C** — This response is specific, states non-retryability plainly, names the conflicting event, and offers concrete alternatives — the full anatomy of a good error.

**10. B** — The module describes a split: harness handles mechanical/transient recovery and enforces budgets, while the model's reasoning is reserved for genuinely ambiguous recovery decisions.

**11. C** — A permission error with no available credential is structural, not transient — retrying identical arguments can never succeed.

**12. B** — Escalation is appropriate once retries/fallbacks are exhausted, the task's premises seem wrong, or unsupervised continuation is itself the risk — not on the first failure, and not never.

**13. B** — MCP standardizes exposing tools, resources, and prompts to any compatible client, replacing bespoke per-system integrations.

**14. B** — A server's job is translating a specific external system's capabilities into the protocol's standard tool/resource/prompt shape.

**15. C** — MCP is plumbing; the model still sees an ordinary name/description/schema tool call, invoked the same structured way as any other tool.

**16. B** — Vetting trust, scoping credentials to least privilege, and auditing actions are the concrete governance controls this module identifies as essential at scale.

**17. B** — `Bash`'s schema is minimal; the actual safety comes from permission prompts, allow/deny lists, and sandboxing around it, not schema-level restriction.

**18. B** — Read, overwrite, and targeted edit are different enough in risk and argument shape that combining them would create the same tangled-schema problem the granularity lecture warns against.

**19. B** — Requiring a prior read prevents a confidently wrong edit made against content the model never actually verified.

**20. C** — The tool set deliberately uses different granularities for different capabilities — broad for the unbounded space of shell commands, narrow for the small, well-defined space of search — showing granularity is a per-capability judgment call, not a fixed policy.

<!-- prevnext:start -->

---

| [&larr; Previous: Key Distinctions - The Tool Layer](14-key-distinctions-the-tool-layer) | [Next: Role Play: Mentoring on Tool Essentials &rarr;](16-role-play-mentoring-on-tool-essentials) |
|:---|---:|

<!-- prevnext:end -->
