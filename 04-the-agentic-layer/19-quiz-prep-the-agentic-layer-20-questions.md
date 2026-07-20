---
title: Quiz Prep: The Agentic Layer (20 Questions)
parent: The Agentic Layer
nav_order: 19
---

# Quiz Prep: The Agentic Layer (20 Questions)

Twenty practice questions covering the Agentic Layer module: the orchestration loop, structured tool use, multi-agent coordination, subagent instantiation, workflow decomposition and safeguarding, and session management. Work through all twenty before checking the Answer Key at the end.

**1.** Which of the following most precisely defines what makes a system "agentic," as distinct from a single LLM call or a fixed pipeline that calls an LLM once?
A. It uses a large, capable model.
B. The model's own decision at each step shapes what it observes next, in a closed loop of decide-act-observe-decide.
C. It produces long, multi-paragraph responses.
D. It has access to more than one tool.

**2.** In the five-step orchestration loop, at which step does the model's output first get classified as either plain text or a tool-use request?
A. Assemble context
B. Call the model
C. Branch on the response
D. Execute the requested tool

**3.** Which of the following is NOT one of the termination conditions discussed for the orchestration loop?
A. The model produces a final text response with no tool-use request.
B. An iteration or turn limit configured by the surrounding system is reached.
C. The model's context window is compressed using a larger model.
D. A human-in-the-loop approval gate declines to let execution continue.

**4.** A system exhibits rapidly increasing per-call cost across a long-running agent loop, even though each individual tool call is cheap. What is the most likely architectural cause?
A. The model is using a more expensive model tier as the loop progresses.
B. Context accumulated from prior tool results is being re-sent in full on every subsequent model call.
C. The tool definitions are malformed.
D. The orchestration loop is running on outdated hardware.

**5.** Why is structured tool use (JSON-Schema-defined function calling) preferred over having the model describe its intended actions in prose?
A. Prose is always longer than JSON, so it costs more tokens.
B. Structured output lets the harness validate arguments against a schema before execution, rather than relying on fragile pattern-matching over free text.
C. Models cannot generate valid JSON reliably.
D. Structured tool use eliminates the need for a system prompt.

**6.** In a Claude API tool definition, what does the `input_schema` field specify?
A. The natural-language description shown to the user.
B. The JSON Schema describing the tool's expected parameters, including types and required fields.
C. The system prompt used for the session.
D. The maximum number of times the tool can be called.

**7.** Why does structured tool use improve composability as a tool set grows?
A. It reduces the total number of tools a system needs.
B. Each tool has a distinct name and schema, so tools can be defined independently and combined without needing bespoke intent-parsing logic to disambiguate them.
C. It automatically deduplicates similar tools.
D. It converts all tools into a single universal function signature.

**8.** A tool result containing thousands of raw search matches is appended verbatim to the conversation and re-sent on every subsequent call. What problem does this illustrate?
A. Structured tool use failure.
B. An orchestration loop with no termination condition.
C. Tool results polluting the context, driving up token cost and crowding out relevant information.
D. A missing approval gate.

**9.** Which of the following is a legitimate architectural motivation for splitting work across multiple agents, according to this module?
A. The task feels complicated.
B. Context isolation, genuine parallelism, specialization, or the need for an independent reviewer.
C. Using multiple agents always looks more sophisticated to stakeholders.
D. It guarantees lower total cost than a single agent.

**10.** From an orchestrator's perspective, how does a subagent dispatch appear, mechanically, within the orchestration loop?
A. As a separate, entirely different API call structure with no relation to tool use.
B. As a structured tool call: a name, arguments, and a returned result appended as a tool result.
C. As a modification to the orchestrator's own system prompt.
D. As a new session entirely disconnected from the orchestrator.

**11.** A workflow requires reviewing three unrelated files for style issues, with no dependency between the reviews. Which coordination pattern is most appropriate?
A. Sequential coordination, to avoid any risk of reconciliation problems.
B. Parallel coordination, since the subtasks are independent and can run concurrently to save wall-clock time.
C. A single agent working through all three serially in one context, since parallelism is never worth the overhead.
D. No agent should touch this task; it should be fully manual.

<br>

**12.** What is the default assumption for how a subagent is instantiated, and why?
A. Shared context, because it gives the subagent full situational awareness.
B. Fresh context, because it isolates the subagent's exploration noise from the orchestrator's own context.
C. Shared context, because it is cheaper in tokens.
D. Fresh context, because subagents cannot access tools otherwise.

**13.** Which of the following should generally be withheld from a fresh-context subagent, according to the module's guidance?
A. A clear task description.
B. Tools irrelevant to its specific subtask and authority beyond the task's scope.
C. A defined reporting format.
D. Relevant context the subagent cannot cheaply rediscover itself.

**14.** A well-formed step in a decomposed workflow should have which of the following properties?
A. It should be as large as possible to minimize the number of steps.
B. It should have a checkable completion condition, a defined input and output, and a coherent scope of tools and context.
C. It should always be executed by a dedicated subagent rather than the orchestrator itself.
D. It should never require human review under any circumstances.

**15.** According to the module, what is a strong signal that a workflow step boundary belongs at a particular point?
A. The step has taken more than five minutes so far.
B. A change in the tools required, the data or context needed, or the reviewer involved.
C. The user asked a new question.
D. The token count of the conversation exceeds a fixed number.

**16.** What distinguishes a checkpoint from a human-in-the-loop approval gate?
A. A checkpoint is any pause for inspection or recovery; an approval gate is specifically a human go/no-go decision, typically reserved for irreversible or high-blast-radius actions.
B. They are the same thing with different names.
C. A checkpoint only occurs at the very end of a workflow; an approval gate can occur anywhere.
D. Approval gates are fully automated, while checkpoints always require a human.

**17.** An agentic workflow is about to permanently delete a dataset with no backup available. Per the module's classification of action types, how should this action be treated?
A. As fully reversible, since version control can restore it.
B. As compensable, since a follow-up action can restore the data.
C. As irreversible, warranting an approval gate before the action rather than a rollback plan after.
D. As low-risk, since deletions are common operations.

**18.** The "just like a server" analogy in this module is used to illustrate which property of a single model call within the orchestration loop?
A. The model call is stateful and remembers all prior sessions automatically.
B. The model call is effectively a stateless request-response cycle: it knows only what's in the context it receives for that call.
C. The model call behaves like a database transaction with automatic rollback.
D. The model call requires a persistent network connection to function.

**19.** A long-running session shows degrading response quality and rising cost over many turns, even though no individual turn is unusually large. What is the most likely cause, and what are the two main mitigation strategies discussed in this module?
A. The model has a hardware fault; restart the server.
B. Unmanaged conversation history growth; mitigate via summarization of older turns or resetting the session.
C. The tool definitions are outdated; mitigate by adding more tools.
D. The user is asking increasingly complex questions; mitigate by simplifying the system prompt.

**20.** Which pair correctly orders the nested timelines introduced in this module from shortest to longest?
A. Session, task, loop iteration, subagent lifespan.
B. Loop iteration, subagent lifespan, task, session.
C. Task, session, loop iteration, subagent lifespan.
D. Subagent lifespan, session, loop iteration, task.

## Answer Key

**1. B** — Agentic behavior is defined by the closed decide-act-observe-decide loop, not by model size, response length, or tool count alone.

**2. C** — Step 3, "branch on the response," is where the harness determines whether the model's output is plain text or a structured tool-use request.

**3. C** — Context compression with a larger model isn't a termination condition for the loop; natural completion, iteration/budget limits, timeouts, and human/error escalation are the actual conditions covered.

**4. B** — Because most implementations re-send the full accumulated history on each call, growing context (not tool cost itself) drives rising per-call cost as the loop progresses.

**5. B** — Structure enables schema validation as an explicit boundary before execution, which prose-based intent detection cannot provide.

**6. B** — `input_schema` is the JSON Schema defining the tool's parameters, types, and required fields.

**7. B** — Distinct names and schemas let independently-defined tools coexist and be disambiguated by the model without custom parsing logic tying them together.

**8. C** — This is the "results polluting the context" problem: unfiltered raw tool output re-sent on every subsequent call.

**9. B** — Context isolation, parallelism, specialization, and independent review are the legitimate motivations; task difficulty alone and perceived sophistication are explicitly called out as insufficient justification.

**10. B** — A subagent dispatch is mechanically identical in kind to any other tool call: name, arguments, and a returned result appended as a tool result.

**11. B** — Independent subtasks with no data dependency are the textbook case for parallel coordination to reduce wall-clock time.

**12. B** — Fresh context is the default specifically because it's what makes context isolation (containing a subagent's exploration noise) actually work.

**13. B** — Irrelevant tools and authority beyond scope should be withheld; a task description, reporting format, and genuinely needed context should be handed over.

**14. B** — Checkable completion, defined input/output, and coherent scope are the stated properties of a well-formed decomposition step.

**15. B** — A change in required tools, data, or reviewer is the module's explicit heuristic for where to place a decomposition boundary.

**16. A** — Checkpoints are the general category (inspection or recovery); approval gates are the specific human-decision subtype, reserved for high-stakes actions.

**17. C** — Irreversible, no-backup deletion is exactly the case the module says needs a gate before the action, since no rollback or compensation is possible afterward.

**18. B** — The server analogy specifically illustrates that each model call is stateless per call, with all "memory" coming from re-sent context rather than the model itself.

**19. B** — Unmanaged history growth is the described cause, with summarization and resetting as the two main mitigation strategies covered.

**20. B** — Loop iteration is shortest, followed by a subagent's lifespan, then a task (which may span several loop iterations and subagent dispatches), then a session (which may contain many tasks).
