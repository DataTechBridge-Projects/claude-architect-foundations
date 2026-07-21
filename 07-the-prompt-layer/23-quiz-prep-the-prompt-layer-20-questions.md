---
title: "Quiz Prep: The Prompt Layer (20 Questions)"
parent: The Prompt Layer
nav_order: 23
---

# Quiz Prep: The Prompt Layer (20 Questions)

Use this as a timed drill: answer all 20 before checking the Answer Key at the end.

1. A prompt asks Claude to "summarize this document." Output varies wildly in length and focus across runs. What is the most likely structural cause?
   A. The model's temperature is set too low
   B. The prompt lacks explicit task framing — no stated audience, purpose, or success criteria
   C. The document is too long for the context window
   D. Few-shot examples were not included

2. Which content generally belongs in the system prompt rather than user content?
   A. The specific document to be summarized on this call
   B. A one-off customer question
   C. Durable behavioral rules and role definition that apply across every call
   D. Raw, untrusted data scraped from a web page

3. Why should untrusted external input generally be kept in clearly demarcated user content rather than woven into system-level instructions?
   A. User content is cheaper to process
   B. System-level content is trusted more heavily, so mixing in untrusted data raises injection risk
   C. System prompts cannot contain variables
   D. User content is always ignored by the model

4. A team wants Claude to consistently write in a very specific "brand voice" that's hard to describe in a few sentences. What technique is best suited to this?
   A. Increasing temperature
   B. Few-shot examples that demonstrate the voice directly
   C. A longer, more detailed prose description of the tone
   D. Schema-constrained generation

5. A few-shot prompt for urgency classification uses three "high urgency" examples that all happen to mention a specific dollar amount. What risk does this introduce?
   A. The model will refuse to classify tickets without dollar amounts
   B. The model may learn "mentions money" as a proxy for urgency, an unintended correlation never stated in the instructions
   C. The prompt will exceed the context window
   D. None — incidental details in examples don't affect model behavior

6. What is the recommended fix for the risk described in question 5?
   A. Remove all examples and rely on instructions alone
   B. Add more high-urgency examples that also mention dollar amounts, to reinforce the pattern
   C. Deliberately include counter-examples that break the spurious correlation (e.g., an urgent case with no dollar amount, a non-urgent case with one)
   D. Increase the temperature setting

7. When selecting few-shot examples, what should they primarily be varied across?
   A. The dimensions along which real production inputs actually vary (length, ambiguity, edge-case status, etc.)
   B. The examples should be as similar as possible to reduce confusion
   C. Only the formatting delimiters between examples
   D. The number of examples, not their content

8. Why is asking nicely for JSON output ("please respond in JSON") an insufficient guarantee for a production pipeline that parses every response programmatically?
   A. Claude cannot produce JSON at all
   B. It's a probabilistic instruction, not a mechanical constraint — the model can still drift (preamble text, wrong field names, stray formatting)
   C. JSON is not supported by the Claude API
   D. It requires a separate fine-tuned model

9. What is the "tool-call-as-output" pattern?
   A. Using a tool definition's schema as the structured-output contract, directing the model to "call" it even if the tool doesn't perform an external action
   B. Asking a tool to generate the prompt itself
   C. Calling multiple tools in parallel to increase output diversity
   D. Using tools exclusively for retrieval, never for output formatting

10. Where should the strongest enforcement of structured output live, according to this module?
    A. In increasingly polite and detailed prompt wording
    B. In post-hoc regex cleanup of the raw text response
    C. As close to the schema/tool-call layer as the platform supports, with format instructions and downstream validation as additional layers
    D. In the system prompt's persona description

11. A prompt asks the model to draft a response and then, within the same generation, critique and revise that draft before returning it. What is the main limitation of this approach compared to a separate reviewer pass?
    A. It costs more tokens than a separate pass
    B. It cannot be used with structured output
    C. The critique inherits the same context, assumptions, and momentum as the draft, so it's poorly positioned to catch errors rooted in those same assumptions
    D. It is not supported by the Claude API

12. What makes a review pass "independent" in the strongest sense?
    A. It uses a higher temperature than the generation pass
    B. It has its own distinct prompt and criteria, and ideally doesn't inherit the generation prompt's exact framing or reasoning
    C. It is run by a different company's model
    D. It always runs before the generation pass

13. Why is an adversarial framing ("find every way this output fails") generally more effective for a reviewer prompt than a softer framing ("does this look okay?")?
    A. Adversarial framing reduces token cost
    B. Softer framings tend to produce agreeable, low-value confirmations rather than genuinely critical evaluation
    C. Adversarial framing is required by the Claude API for tool use
    D. There is no meaningful difference between the two framings

14. A low-stakes, high-volume internal summarization task, easily corrected by a human skimmer, is being considered for a full independent multi-pass review pipeline. What's the best judgment call?
    A. Always add the heaviest review available, since more verification is always better
    B. Skip verification entirely regardless of stakes
    C. Weigh the added cost and latency against the low cost of an occasional miss — heavyweight review is likely not justified here
    D. Multi-pass review is mandatory for all production prompts

15. What is a rubric, in the context of quality and feedback enforcement?
    A. A single vague instruction to "do good work"
    B. A list of specific, checkable criteria the output should satisfy, enabling structured self- or independent evaluation
    C. A synonym for temperature setting
    D. A required field in every tool schema

16. In a multi-step pipeline, why is catching a low-quality output early (right after it's produced) more valuable than catching it several steps later?
    A. Early catches are always cheaper computationally regardless of context
    B. Later steps often treat earlier output as trusted ground truth, so an early error can compound and become harder to trace and fix downstream
    C. It isn't more valuable — timing doesn't matter
    D. Only human reviewers can catch errors, so timing is irrelevant to automated checks

17. What does temperature control, and what setting is generally recommended as a default for batch extraction/classification tasks?
    A. Output length; higher is recommended for batch tasks
    B. Randomness in token selection; lower temperature is generally recommended for consistency across a batch
    C. The number of tool calls allowed per request; lower is recommended
    D. Context window size; higher is recommended

18. A batch run performs well on the first several hundred items but quietly degrades on a later cluster of inputs from a newly launched product line. What is this an example of?
    A. Temperature drift
    B. A schema validation failure
    C. Distribution shift — the prompt wasn't validated against the full range of inputs the batch actually contained
    D. A tool-calling error

19. Why is spot-checking only the first few outputs of a batch run an insufficient monitoring strategy?
    A. The first outputs are always the least accurate
    B. It cannot catch drift or degradation that emerges partway through the run on a different input slice
    C. Spot-checking is never useful in any context
    D. It uses too much of the context window

20. What is the core idea behind treating a prompt as a "container" for a workflow step, as discussed in this module?
    A. Prompts should be as long as possible to include maximum context
    B. Each prompt should have a defined interface — explicit inputs and outputs — without silently depending on hidden state from other steps
    C. Prompts should always be run inside literal Docker containers for security
    D. Containers and prompts are unrelated concepts with no useful analogy

## Answer Key

1. B — Vague framing (no audience, purpose, or success criteria) is the most common cause of inconsistent open-ended output.
2. C — Durable, session-spanning rules belong in the system prompt; per-call specifics belong in user content.
3. B — System-level instructions are trusted more heavily by the model, so mixing in untrusted data raises injection risk.
4. B — Tone/voice is a "you know it when you see it" property that examples demonstrate far more precisely than prose description.
5. B — A shared incidental detail across examples can be learned as an unintended proxy for the real target.
6. C — Deliberately breaking the spurious correlation with counter-examples prevents the model from keying off the wrong feature.
7. A — Examples should span the real dimensions of variation in production inputs, not be minor variations of one type.
8. B — Prompt wording describes a likely output distribution; it doesn't mechanically guarantee conformance the way schema enforcement does.
9. A — Tool-call-as-output repurposes a tool's validated input schema as the structured-output contract.
10. C — Enforcement should be layered, with the strongest constraint as close to the schema/tool-call mechanism as possible.
11. C — Inline self-critique shares the same context and assumptions as the original draft, limiting what it can catch.
12. B — True independence comes from separate criteria and limited inheritance of the generator's own framing and reasoning.
13. B — Soft framings bias toward leniency; an explicit adversarial mandate produces more genuinely critical review.
14. C — Verification level should be proportional to the cost of a miss; low-stakes, human-correctable tasks rarely justify heavy review.
15. B — A rubric is a set of specific, checkable criteria used for self- or independent evaluation.
16. B — Downstream steps often trust upstream output uncritically, so errors compound and become harder to trace the later they're caught.
17. B — Temperature controls randomness in generation; low temperature is the default for consistency in batch tasks.
18. C — A later cluster of inputs unlike the validated distribution is a textbook case of distribution shift causing silent drift.
19. B — Drift and degradation can emerge partway through a run, which early-only spot-checking will miss entirely.
20. B — A well-designed prompt step declares explicit inputs and outputs rather than leaking or depending on hidden state, just like a container.

<!-- prevnext:start -->

---

| [&larr; Previous: Key Distinctions - The Prompt Layer](22-key-distinctions-the-prompt-layer) | [Next: Role Play: Mentoring on Prompting Essentials &rarr;](24-role-play-mentoring-on-prompting-essentials) |
|:---|---:|

<!-- prevnext:end -->
