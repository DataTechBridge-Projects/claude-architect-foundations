---
title: Cross-Layer Patterns
parent: Additional Topics
nav_order: 2
duration: "13:12"
---

# Cross-Layer Patterns
{: .no_toc }

*Read duration: 13:12*

Study the five layers in isolation for long enough and it's easy to start treating them as five unrelated checklists — Agentic has its rules, Tool has its rules, and so on down the stack. But the strongest architects, and the exam's hardest scenario questions, operate at a different altitude: recognizing that a handful of design patterns recur across multiple layers at once, expressed each time in that layer's own vocabulary but solving the same underlying problem. Spotting one of these patterns is often faster than reasoning a scenario through from scratch, because once you recognize which pattern is in play, you already know its shape — what it protects against, what it costs, and where it tends to get applied badly. This lecture names six of them.

| Pattern | Layers It Spans | Core Idea |
|---|---|---|
| 1. The Do-Nothing Option | Context, Agentic, Tool, Workflow | Make not acting, or acting minimally, a legitimate first-class option |
| 2. Structure Over Prose | Tool, Prompt | Default to structured, validatable output; reserve free text for genuine judgment |
| 3. Checkpoints and Gates at the Right Altitude | Agentic, Workflow, Context | Match the kind of pause to what it actually needs to catch |
| 4. Granularity Matched to Stakes | Agentic, Tool, Context, Workflow | Match grain size to cost-of-error and predictability, never fix it |
| 5. Provenance and Trust Travel With Information | Context, Tool, Prompt | Trust is a property of content plus its origin, not content alone |
| 6. Justify Complexity Against a Named Unpredictability | Agentic, Workflow, Prompt | Withhold added machinery until you can name what it's there to handle |

## Pattern 1: The Do-Nothing Option

Every layer eventually confronts the same question: what happens when the system doesn't have enough information, or enough confidence, to act? The tempting design is to force a decision anyway, because "do something" always feels more productive than "do nothing." The recurring correct answer, across layers, is to make *not acting* — or acting minimally, by asking — a legitimate, first-class option rather than an edge case the design forgot to handle.

At the **Context layer**, this is the escalation spectrum you studied directly: guess-and-continue is almost never right, and the legitimate responses — flag an assumption, ask a clarifying question, escalate, or fully stop — all involve some version of not proceeding on incomplete information. At the **Agentic layer**, it shows up as the stopping condition built into the loop itself: an agent needs a real, reachable path to "I don't have what I need to finish this" rather than only "keep calling tools until something looks like an answer." A loop with no legitimate exit besides a final answer will manufacture one, confidently, whether or not it's correct.

The pattern touches the **Tool** and **Workflow** layers too, just less centrally. A well-designed tool can decline — return a structured "no valid match, here's why" instead of forcing a best-guess result — and a workflow's approval gate is, structurally, a do-nothing option promoted to a formal checkpoint. Wherever you see this pattern tested, the wrong answer is almost always the option that keeps the system moving forward on a guess; the right answer is whichever legitimate non-action is narrowest and cheapest for the specific stakes involved.

## Pattern 2: Structure Over Prose, Prose as Escape Hatch

Both the Tool and Prompt layers converge on the same design instinct: default to structured, validatable representations, and reserve free text for the places where genuine judgment is needed and pre-enumerating the space would hurt more than help.

At the **Tool layer**, this is the structure-versus-prose distinction within a schema — typed fields, enums, and ranges for anything with a known, finite set of valid values, with a free-text field reserved for genuinely open-ended input the model needs judgment to fill in. At the **Prompt layer**, the same instinct shows up as the gap between format instructions and schema enforcement: politely asking for JSON is a request the model can drift from under pressure, while schema-constrained generation or a validated tool-call interface enforces the shape at the mechanism level rather than the wording level. The unifying idea is that **structure is a reliability mechanism, not a stylistic preference** — every field or output slot you leave as free text is a place where downstream parsing, tool selection, or automation can silently break, and every field you over-structure is a place where the model's legitimate judgment gets squeezed out.

The pattern's failure mode runs in both directions. Under-structuring produces output that "usually" parses, the same way a prompt that "usually works" is a liability in production. Over-structuring — cramming a field that genuinely needs open-ended reasoning into a rigid enum — produces technically valid output that's substantively wrong, because the model had no legitimate way to express what the situation actually called for. Exam questions in this space often present a schema or prompt that's failed in a specific, describable way, and ask you to identify whether the fix is "add more structure" or "loosen this one field back to prose" — the two are opposite fixes to two different failure shapes that look similar on the surface.

## Pattern 3: Checkpoints and Gates at the Right Altitude

Nearly every layer eventually needs a deliberate pause point, but the layers disagree, usefully, on what that pause is for — and conflating the different kinds of pauses is one of the most common design mistakes the exam probes.

The **Agentic layer** distinguishes a checkpoint (any pause for inspection or recovery) from an approval gate (specifically a human go/no-go decision, reserved for irreversible or high-blast-radius actions). The **Workflow layer** operationalizes that same idea as validation and review gates placed at points where the cost of an error is high, and as CI/CD integration, which gives a Claude-driven step the same gating, logging, and rollback discipline as any other pipeline stage. The **Context layer** adds the sharper, more testable version of the same idea: a review checkpoint that exists on paper is not the same as one that functions, and a gate only functions if it shows the actual decision (not the whole session transcript), surfaces uncertainty explicitly, and offers response options more granular than a binary approve or reject.

The pattern's recurring failure isn't the absence of a gate — architects rarely forget gates entirely — it's placing the *wrong kind* of gate, or placing a gate at the wrong altitude. A checkpoint built to inspect intermediate work gets treated as if it were an approval gate protecting an irreversible action, and nobody notices until something irreversible slips through unreviewed. Or a gate is defined but scoped so poorly — dumping an entire session log in front of an overworked reviewer — that it exists in the architecture diagram without ever catching a real error. When a scenario describes a review step that "didn't work," your first move should be asking which of these two failures actually occurred: no gate, or a gate that couldn't function given what it was shown.

{: .important }
> Don't conflate a **checkpoint** (any pause for inspection or recovery) with an **approval gate** (specifically a human go/no-go decision reserved for irreversible or high-blast-radius actions). Placing the wrong kind of pause — or the right kind at the wrong altitude — is one of the exam's most common trap scenarios.

## Pattern 4: Granularity Matched to Stakes, Never Fixed

Four separate layers each contain their own version of a granularity question, and all four resolve it the same way: match the grain size to how well-understood and how costly-to-get-wrong the specific unit is, rather than adopting one fixed granularity as a house style.

The **Agentic layer** frames this as coarse-grained versus fine-grained task decomposition — coarse steps mean fewer handoffs but push more implicit decomposition burden onto whatever executes each step, while fine steps are independently checkpointable but multiply coordination overhead. The **Tool layer** frames the identical trade-off as tool granularity: narrow, near-duplicate tools multiply disambiguation burden as a set grows, while one broad, overloaded tool pushes that same burden into argument-filling instead. The **Context layer** applies it to information selection — file-level context preserves surrounding detail at higher token cost, symbol-level extraction is cheaper but risks stripping context that clarified correct usage — and to escalation itself, where the right response to ambiguity is a granularity judgment (a narrow clarifying question versus a full stop) matched to stakes rather than one default response applied everywhere. The **Workflow layer** applies it at the level of an entire process: some steps get pinned into a fixed pipeline, others are left to agentic sequencing, chosen individually per step rather than for the process as a whole.

What makes this a genuine cross-layer pattern rather than four coincidentally similar ideas is that the underlying variable is the same in every case: how much does getting this specific unit wrong actually cost, and how well can its correct handling be specified in advance? Whenever an exam question asks you to choose "how big should this unit be" — a tool, a decomposed subtask, a context slice, a workflow step — that's the question underneath it, and there is deliberately no universally correct answer independent of the specific case.

## Pattern 5: Provenance and Trust Travel With the Information

A recurring, easy-to-miss failure across layers is treating information as equally trustworthy just because it's present, rather than attaching — and preserving — metadata about where it came from and how much confidence it deserves.

The **Context layer** names this directly as provenance tracking: knowing where a fact in context originated, how current it is, and what to do when two sources disagree, with recency and authority as different (and sometimes conflicting) signals for resolving that disagreement. The **Tool layer** carries the same concern in its error-response design: a designed error explicitly states whether a failure is retryable, which is exactly a trust signal about the result just returned, while a raw stack trace passthrough carries no such signal and leaves the model to guess. MCP governance extends this further — a connected MCP server is a dependency with real system access, and knowing which servers are trusted, what scope they're credentialed for, and whether their actions are auditable is provenance tracking applied to entire tool ecosystems rather than individual facts. The **Prompt layer** encodes a simpler, structural version of the same idea in the system-versus-user content boundary: system content is trusted more heavily than user content precisely because of where it came from, and collapsing that distinction — especially with untrusted input — is a security posture failure, not just a style mistake.

The unifying principle is that **trust is not a property of content alone; it's a property of content plus its origin**, and that origin has to be tracked and carried forward, not silently discarded the moment information enters context or a prompt. A tool result that looks like clean ground truth but was actually filtered by an undisclosed default, or a fact repeated so many times in a session that it reads as verified when it was never re-checked, are the same underlying failure wearing different layers' clothing.

## Pattern 6: Justify Complexity Against a Named Unpredictability

Finally, a pattern about restraint itself: across the Agentic, Workflow, and Prompt layers, the default architectural posture is to withhold complexity — more autonomy, more agents, more verification passes — until you can name, specifically, what unpredictable branch or specialized need the added machinery is there to handle.

The **Agentic layer** states this as the preference for the simplest architecture that works: a single well-prompted call beats a workflow, a fixed workflow beats an autonomous agent, and a single agent beats a multi-agent system, in each case unless something specific about the task's unpredictability or need for specialized, isolated work justifies the next level up. The **Workflow layer** applies the same restraint one level down, at the step level rather than the system level: a step is deterministic if its correct behavior can be fully specified in advance, and only genuine ambiguity or context-dependence — not mere complexity — earns it agentic treatment. The **Prompt layer** applies the identical logic to verification: more checking passes are never free, and the right amount of verification is whatever is proportional to what an undetected failure would actually cost, not a fixed ritual applied to every output regardless of stakes.

This pattern is the connective tissue behind a huge fraction of exam distractors, because the wrong answer choice in these scenarios is almost never *incorrect* on its own terms — a multi-agent system, an autonomous step, or an extra review pass will usually *work*. It's simply more than the scenario's actual risk profile justifies, and the tell is always the same: if you can't name the specific unpredictable branch or specialized capability the extra machinery exists to handle, you probably don't need it yet.

## Why It Matters for the Exam

These six patterns are worth memorizing by name, not just by content, because recognizing "this is a do-nothing-option question" or "this is a granularity question" is frequently faster than working a scenario from first principles under time pressure. When you're stuck between two plausible answers on a scenario question, ask whether one of these six patterns is quietly the axis the question is actually built around — more often than not, it is.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Intro](01-module-intro) | [Next: Common Antipatterns &rarr;](03-common-antipatterns) |
|:---|---:|

<!-- prevnext:end -->
