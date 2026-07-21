---
title: "Quiz Prep: The Context Layer (20 Questions)"
parent: The Context Layer
nav_order: 24
---

# Quiz Prep: The Context Layer (20 Questions)

Twenty questions covering this module: context composition, summarization and compaction, isolation, escalation and ambiguity, error propagation, large-codebase retrieval, human review design, and data provenance. Work through all twenty before checking the answer key.

1. A team keeps expanding its `CLAUDE.md` because "more context can only help." What is the most likely negative consequence?
   A. `CLAUDE.md` content cannot be cached, so this always increases latency
   B. It has no real downside since `CLAUDE.md` is only loaded once per project
   C. It becomes a standing cost paid on every turn, edging out other relevant context through signal dilution
   D. It will exceed the hard token limit within a single conversation

2. Which of the following is a defining difference between summarization and compaction as covered in this module?
   A. Summarization is reversible; compaction is not
   B. Summarization rewrites content into a shorter narrative preserving decisions and justification; compaction removes redundant or superseded material while preserving structure
   C. Compaction only applies to tool results; summarization only applies to conversation history
   D. They are two names for the same technique

3. An agent is given thirty tools "just in case" they're needed, even though most tasks only use two or three of them. What is the primary context management cost of this design?
   A. None, because unused tool definitions are automatically excluded from the window
   B. Every tool's name, description, and schema costs tokens on every turn the tool set is in scope, regardless of whether it's called
   C. It only matters if the tools have overlapping names
   D. It increases the cost of `CLAUDE.md` loading specifically

4. What is the main risk of milestone-based checkpointing compared to threshold-based compaction?
   A. It requires no judgment about task structure, so it's less flexible
   B. It cannot be combined with summarization
   C. It is unrelated to context management entirely
   D. If milestones are placed poorly, compression may not happen until well past the point the window is under real pressure

5. Why does prompt caching favor placing system instructions, `CLAUDE.md`, and tool definitions early and keeping them stable, while placing live conversation and fresh retrieval later?
   A. Caching has no relationship to context ordering
   B. Stable content placed early and kept unchanged benefits from a reusable cached prefix, while volatile content downstream doesn't break that cache
   C. Only user messages can be cached
   D. Tool definitions must always appear after conversation history for the API to parse them

6. In a multi-agent system, why is it generally preferable for a subagent to return a clean, purpose-built summary rather than its full raw transcript to the orchestrator?
   A. Raw transcripts are not technically permitted by the API
   B. It prevents the subagent from being reused later
   C. It avoids flowing exploratory noise and dead ends back into the orchestrator's context, which would dilute its working set
   D. Summaries are always more accurate than the underlying transcript

7. A support agent encounters an instruction that could reasonably mean either "the customer's billing address" or "the customer's shipping address," with nothing in context to disambiguate. What is the most defensible response?
   A. Update whichever address was mentioned most recently in the conversation, without comment
   B. Pick the billing address by default, since it's more commonly relevant
   C. Ask a narrow, specific clarifying question about which address is meant before proceeding
   D. Update both addresses to avoid the risk of choosing wrong

8. Why is "guess and continue" considered especially risky in agentic systems, more so than in typical human workflows?
   A. Agents are legally prohibited from making assumptions
   B. A guess that gets acted on becomes indistinguishable from a verified fact in later context, and autonomous agents often lack the ambient human checks that catch bad guesses quickly
   C. Guessing always produces lower-quality prose than asking
   D. It is only risky when the agent is operating without any tools at all

9. An agent processing fifty invoices finds that three have an ambiguous vendor match. What is the best-granularity response?
   A. Stop processing all fifty invoices until a human resolves the three ambiguous cases
   B. Guess the vendor match for the three ambiguous invoices to avoid interrupting the batch
   C. Complete the forty-seven clear invoices and escalate the three specific ambiguous cases with relevant context attached
   D. Skip the three ambiguous invoices silently and report only the forty-seven completed ones

10. What is the primary danger of a system that escalates every minor uncertainty to a human reviewer?
    A. It technically violates the granularity principle by having too many options
    B. It has no real downside as long as reviewers are paid for their time
    C. It trains reviewers to stop reading escalations carefully, undermining the value of escalation when it actually matters
    D. It makes the agent's context window grow uncontrollably

11. Why can a propagated error be harder to detect than a single, isolated wrong answer?
    A. Propagated errors always involve incorrect arithmetic, which is harder to spot
    B. The flawed premise gets buried under several layers of individually reasonable-looking decisions, so the final output looks well-reasoned even though it's wrong
    C. Propagated errors only occur in multi-agent systems, which are inherently harder to audit
    D. Single wrong answers are actually more common, so reviewers are less prepared for them

12. How can aggressive, poorly designed summarization make error propagation worse rather than better?
    A. It cannot — summarization only ever reduces risk
    B. It always removes the correct facts and keeps the incorrect ones
    C. It can strip away caveats and uncertainty markers, leaving a shaky inference presented with the same unearned confidence as a verified fact
    D. It increases the size of the context window, which increases risk

13. What is the purpose of a checkpoint, as distinct from a milestone-based compaction boundary, even though the two often coincide?
    A. A checkpoint saves tokens; compaction re-examines conclusions
    B. A checkpoint re-examines whether current conclusions still deserve to be treated as true; compaction focuses on what can be compressed
    C. They are identical in purpose and only differ in naming
    D. A checkpoint only applies to human reviewers, never to the agent itself

14. When is re-verification (actively re-confirming a fact against ground truth) most worth its cost?
    A. For every fact in context, regardless of stakes, to maximize accuracy
    B. Only at the very end of a session, right before the final output is delivered
    C. For load-bearing conclusions about to justify an expensive or hard-to-reverse action
    D. Never — if a fact is already in context, it should be treated as settled

15. A large, actively developed codebase is too big to fit in any context window. What is the primary shift in strategy this requires compared to a small, stable project?
    A. The focus moves from deciding what to trim from an already-in-scope set to deciding what narrow slice to retrieve into scope in the first place
    B. `CLAUDE.md` becomes unnecessary and should be removed entirely
    C. Summarization and compaction are no longer useful techniques
    D. The codebase should be split into many separate unrelated repositories

16. A developer needs to find every location in the codebase that calls a specific function, starting from one known usage. Which retrieval technique is best suited to that specific need?
    A. Keyword search alone
    B. Structural/symbol-level navigation using the call graph
    C. Semantic/embedding-based search
    D. Loading the entire repository into context at once

17. Why would building an elaborate semantic-search-and-symbol-extraction retrieval pipeline be the wrong choice for a small, stable, single-purpose internal tool?
    A. Semantic search cannot be used on small codebases for technical reasons
    B. It adds real engineering complexity and latency to solve a scale problem that project doesn't actually have; a lean, always-loaded `CLAUDE.md`-style approach already fits
    C. Small codebases are not permitted to use retrieval by convention
    D. It would make the code less readable to humans

18. A human review checkpoint hands the reviewer the full twenty-turn transcript of an agent's session instead of the specific facts relevant to the decision at hand. What failure mode does this risk?
    A. None, since more information always improves human decisions
    B. The same signal-dilution problem that affects models, causing the reviewer to skim or take far longer than the checkpoint's value justifies
    C. It is only a problem if the transcript exceeds the reviewer's screen size
    D. It guarantees the reviewer will reject the request

19. Two sources in context disagree: a system-of-record lookup says a customer's account is active, and a two-week-old email in the conversation says the customer requested cancellation. What is the best general approach to resolving this kind of conflict?
    A. Always trust whichever source is more recent in the conversation, without exception
    B. Always trust whichever source sounds more detailed or confident
    C. Weigh source authority, currency, and verifiability together, and escalate or flag explicitly if the conflict doesn't resolve clearly
    D. Average the two claims into an uncertain middle state and proceed

20. Which best describes the purpose of attaching "source, timestamp, and trust level" metadata to a fact in context, as covered in the provenance lecture?
    A. It is a formatting convention with no bearing on agent behavior
    B. It exists solely to satisfy audit logging requirements unrelated to reasoning quality
    C. It gives every downstream step — checkpoints, re-verification, human review, conflict resolution — the information needed to judge how much a claim deserves to be trusted
    D. It is only relevant for regulated industries like finance and healthcare

## Answer Key

1. **C** — An always-loaded file's cost is paid on every turn regardless of relevance, and excess content dilutes the model's attention on what actually matters.
2. **B** — Summarization is an editorial rewrite preserving decisions/justification; compaction is closer to mechanical removal of redundant material.
3. **B** — Tool definitions cost tokens on every turn they're in scope, whether or not the tool is ever invoked that turn.
4. **D** — Poorly placed milestones can leave compression lagging behind actual context pressure, unlike threshold-based triggers tied directly to budget.
5. **B** — Caching rewards a stable, unchanging prefix; volatile content belongs downstream of it so edits don't invalidate the cache.
6. **C** — Returning a clean summary keeps exploratory noise contained to the subagent rather than polluting the orchestrator's working set.
7. **C** — A narrow clarifying question resolves genuine ambiguity cheaply without guessing or silently picking one interpretation.
8. **B** — Guesses that get acted on look identical to verified facts afterward, and autonomous agents often lack the ambient human correction that catches bad guesses quickly.
9. **C** — Granular escalation isolates just the ambiguous cases rather than halting unrelated, unambiguous work or silently guessing/skipping.
10. **C** — Constant escalation causes alert fatigue, training reviewers to skim and undermining escalation's value when it's genuinely needed.
11. **B** — The compounding of individually reasonable steps camouflages the flawed premise, making the final output look well-reasoned despite being wrong.
12. **C** — Summaries that drop caveats and uncertainty markers make shaky inferences read as confidently as verified facts.
13. **B** — A checkpoint is about re-examining whether conclusions still hold; compaction is about reclaiming space, even though they often align at the same point in a task.
14. **C** — Re-verification is most valuable where a conclusion is both load-bearing and about to justify a costly, hard-to-reverse action.
15. **A** — Large codebases shift the problem from subtractive trimming of in-scope material to additive, selective retrieval into scope.
16. **B** — Structural/symbol-level navigation follows real code relationships like call graphs, which is exactly what's needed to find all callers of a known function.
17. **B** — For a small, stable project, the complexity of a retrieval pipeline isn't justified; a lean static approach solves the same problem more simply.
18. **B** — An unfiltered, lengthy transcript causes the same signal-dilution problem in a human reader that it would in a model, degrading review quality.
19. **C** — Conflicts should be resolved by weighing authority, currency, and verifiability together, and surfaced rather than silently guessed at when unresolved.
20. **C** — Provenance metadata is what allows every other technique in the module — checkpoints, re-verification, human review, conflict resolution — to actually function.

<!-- prevnext:start -->

---

| [&larr; Previous: Key Distinctions - The Context Layer](23-key-distinctions-the-context-layer) | [Next: Role Play: Mentoring on Context Essentials &rarr;](25-role-play-mentoring-on-context-essentials) |
|:---|---:|

<!-- prevnext:end -->
