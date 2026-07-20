---
title: Workflow Design
parent: The Workflow Layer
nav_order: 6
duration: "13:10"
---

# Workflow Design
{: .no_toc }

*Video duration: 13:10*

A **workflow**, in this course's vocabulary, is the concrete sequence of steps that gets a piece of work done — regardless of whether any individual step involves an LLM at all. Workflow design is the discipline of laying that sequence out deliberately: what happens first, what happens next, what has to succeed before the following step is even attempted, and — the question this lecture spends the most time on — which of those steps are deterministic and fixed versus which are handed to an agent's judgment.

This is a different question from the one the Agentic layer asks. Agentic architecture asks the big, one-time design question: should this *entire* piece of work be a single agent, a fixed pipeline with no agentic judgment anywhere in it, or a multi-agent system? Workflow design operates one level down from that answer, inside whatever architecture you picked: given that you're building a pipeline (or an agent, or a multi-agent system), how exactly is the work inside it broken into steps, and what governs each one?

## Sequencing: Decomposing Work Into Steps

The first job of workflow design is decomposition — breaking one large, vague task ("update the documentation when the API changes") into a concrete sequence of smaller, well-defined steps ("detect which endpoints changed" → "identify which doc pages reference them" → "draft updated content" → "check the draft against the style guide" → "open a PR" → "wait for human review"). This isn't busywork; the quality of that decomposition determines almost everything else about how the workflow behaves, because each step becomes an independent unit you can reason about, test, constrain, and monitor on its own.

Good decomposition tends to isolate points where something distinct is actually happening: a change of representation (raw diff → structured list of changed endpoints), a change of *who* or *what* is deciding (an automated check → a human approval), or a natural checkpoint where failure should stop the pipeline rather than propagate forward with bad input. Bad decomposition either collapses too much into one step (an enormous, vaguely-specified "do the whole thing" step that's hard to constrain, test, or debug) or fragments too finely (dozens of trivial steps that add coordination overhead without adding any real safety or clarity).

Sequencing also has to account for dependencies and failure paths, not just the happy path. What happens if step two produces nothing usable — does the workflow retry, escalate to a human, or fail the whole run cleanly? A workflow design that only specifies the success path is an incomplete design; the exam consistently tests whether candidates have thought about the failure branches, not just the intended one.

## Deterministic Steps vs. Agentic Steps

The central judgment call in workflow design — and the one most directly tested on the exam — is deciding, step by step, whether a given piece of work should be **deterministic** (fixed logic, the same procedure every time, no model judgment involved) or **agentic** (handed to a model that reasons about the specific situation and decides what to do).

Deterministic steps are the right choice when the work is well-understood, repeatable, and doesn't actually benefit from judgment — it benefits from consistency. Running a test suite, computing a diff, validating that a file matches a schema, formatting code, computing a checksum: these are steps where the "right" outcome is fully specified by the input, and inserting a language model into that step would only add cost, latency, and a new source of nondeterministic failure without adding any value. A rule of thumb worth internalizing: if you could write the step as a deterministic script and it would do the job correctly every time, write it as a deterministic script. Don't spend a model call reasoning your way to an answer a regex or a linter already gives you for free.

Agentic steps earn their place when the work genuinely requires judgment that can't be fully enumerated in advance — interpreting an ambiguous bug report, deciding how to phrase a customer-facing explanation, weighing which of several valid refactoring approaches best fits the surrounding code's existing style, or synthesizing a summary from sources that don't all agree. These are steps where the "correct" output depends on context and reasoning, not on a fixed rule, and where trying to hand-code every possible case in advance would be brittle or simply impossible.

A useful diagnostic when you're unsure which category a step falls into: ask whether you could write a complete, correct specification of the step's logic without knowing anything about the *specific instance* of work in front of it. If yes — if the specification is genuinely input-independent — it's a deterministic step, even if writing that specification is itself difficult. If no — if the right behavior depends on details you can't anticipate until you see the actual input — it's a genuine candidate for an agentic step.

It's also worth resisting a subtler trap: a step can *look* like it needs judgment because it's complicated, when it's actually just complex-but-fully-specified. Tax calculations, currency conversions, and complicated-but-well-defined business rules are not judgment calls just because they have a lot of branches — they're deterministic logic that happens to be intricate. Don't hand a model a task because it's hard; hand it a task because it's genuinely ambiguous or context-dependent.

## Mixed Workflows Are the Common Case

In practice, almost no real workflow is purely one or the other. The far more common shape is a pipeline that alternates: a deterministic step gathers and structures data, an agentic step reasons over that structured data and produces a judgment call or a draft, another deterministic step validates the output against hard constraints (does it match a schema, does it pass the existing test suite, is it within a length limit), and — often — a human checkpoint sits at the highest-stakes transition before anything ships.

This alternation is a design pattern worth naming: **deterministic steps bound and validate agentic steps.** A deterministic pre-step narrows what the agentic step actually has to reason about (feeding it a clean, structured diff rather than "go figure out what changed"), and a deterministic post-step catches outputs that violate hard constraints before they propagate further (a schema check, a test run, a linter pass). This is one of the most reliable ways to get the benefits of agentic judgment — flexibility, the ability to handle cases you didn't fully anticipate — without accepting its full downside, which is unconstrained, unpredictable output. The agent gets exactly the room to exercise judgment where judgment is actually needed, bounded tightly everywhere else.

## Workflow Structure and the Agentic Layer's Single-Agent/Multi-Agent Decision

Workflow design and the Agentic layer's architecture decision aren't independent — they shape each other, and the exam frequently probes the seam between them.

A workflow with clearly separated, well-bounded steps is often exactly what makes a *fixed pipeline* (no agentic judgment governing the flow between steps, even if individual steps are agentic) a viable and preferable choice over a single autonomous agent freely deciding what to do next. If you can specify the sequence in advance — step one always precedes step two, which always precedes step three — you don't need an agent deciding *that* sequence at runtime; you need a fixed orchestration layer that calls each step in order, some of which happen to invoke a model. This is cheaper, faster, and dramatically easier to audit than a single agent reasoning its way through the same sequence turn by turn, because the control flow is visible in the workflow definition itself rather than implicit in a model's decisions.

Conversely, a workflow whose step boundaries genuinely can't be pinned down in advance — where what needs to happen next depends heavily on what was just discovered — is a signal that you may actually need agentic control over the *sequencing itself*, not just agentic judgment within individual steps. That pushes you toward a single autonomous agent (or, if the task also benefits from specialized decomposition, a multi-agent system) rather than a fixed pipeline, because a fixed pipeline can't adapt its own step order to information it doesn't have until runtime.

Multi-agent decisions interact with workflow design too: if your workflow decomposition reveals a step that requires meaningfully different expertise, context, or tooling than the rest of the pipeline — and where the coordination cost of a separate agent is worth paying — that's often the moment where a workflow step becomes a candidate for a subagent, formalizing what was already a natural seam in the decomposition into an actual architectural boundary.

**Why it matters for the exam:** Scenario questions in this domain typically describe a multi-step business process and ask you to classify individual steps as deterministic or agentic, or ask you to justify why a fixed pipeline is preferable to a single freewheeling agent (or vice versa) for a described process. The reliable approach is the same each time: identify whether each step's correct behavior can be fully specified in advance independent of the specific input, and check whether the overall sequence of steps is itself predictable or genuinely dependent on runtime discovery. Getting both of those judgments right is what workflow design is actually testing.
