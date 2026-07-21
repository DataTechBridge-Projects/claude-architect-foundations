---
title: Batch Processing
parent: The Prompt Layer
nav_order: 10
duration: "17:22"
---

# Batch Processing
{: .no_toc }

*Video duration: 17:22*

A prompt that works well once, on one input, tested by hand, is a proof of concept. A prompt that works well the same way across ten thousand different inputs, unattended, is a production system — and getting from the first to the second is a distinct engineering problem with its own failure modes. **Batch processing**, in this context, means running the same prompt (or the same prompt template, filled in differently) across a large number of inputs, and it surfaces reliability issues that a single hand-tested example never reveals, because a single example can't expose variance, drift, or the long tail of inputs you didn't think to test.

## Consistency Across a Batch

The first thing batch processing exposes is variance you didn't know you had. When you test a prompt on one input and read the result yourself, you unconsciously fill in gaps and forgive minor inconsistencies — a slightly different phrasing here, a slightly different level of detail there — because you're reading with the intent of understanding, not the intent of comparing. Run the same prompt across five hundred similar inputs and lay the outputs side by side, and inconsistency that was invisible in isolation becomes obvious: some outputs use one field-naming convention and some use another, some include an optional section and some omit it without a clear reason, some interpret an ambiguous instruction one way and others interpret it the opposite way.

This matters because batch outputs are frequently consumed as a set, not individually. If a batch of five hundred extracted records feeds into a downstream aggregation or report, a parser or a reader needs every record to follow the same schema and the same conventions — not "mostly the same," but actually the same, because "mostly" is exactly where automated downstream processing breaks. Achieving that level of consistency is a direct extension of everything earlier in this module: explicit success criteria, unambiguous instructions, and schema-constrained structured output all reduce variance, and a batch is where the payoff (or the cost of skipping them) becomes visible at scale rather than staying hidden in a handful of manually reviewed examples.

A practical technique for surfacing consistency problems before they reach production is to run a prompt against a deliberately diverse sample batch — one that spans the range of real input variation discussed in the few-shot lecture — and then examine the *outputs as a set*, specifically looking for unexplained divergence between similar inputs. If two inputs that are substantively similar produce structurally different outputs, that's a signal the prompt has an ambiguity or an underspecified decision point that's being resolved differently from run to run, and it's worth tightening before the prompt goes anywhere near real volume.

## Sampling and Temperature Considerations

Claude's API exposes a **temperature** parameter that controls how much randomness is injected into token selection during generation — lower temperature biases the model toward its highest-probability continuation at each step, producing more deterministic, repeatable output, while higher temperature allows more variation and, with it, more creativity but also more variance run to run on the same input.

For most batch-processing use cases — extraction, classification, structured data transformation, anything where consistency and predictability matter more than creative variation — a low temperature setting is the right default, precisely because the goal of a batch run is usually to apply the *same judgment* consistently across many inputs, not to generate creative diversity across them. Higher temperature has real uses (brainstorming variations, generating diverse creative drafts, exploring a solution space), but those uses are the exception in batch processing, not the norm, and reaching for a low temperature by default for batch work is a sound baseline that should require a specific reason to override.

It's worth being precise about what temperature does and doesn't buy you, though: even at temperature zero, output isn't guaranteed to be bit-for-bit identical across repeated calls with identical input, due to other sources of nondeterminism in how large models are served at scale. Low temperature substantially reduces variance; it doesn't eliminate it as an architectural guarantee. If a use case genuinely requires exact reproducibility — the same input must always produce the exact same output, byte for byte — that requirement usually needs to be satisfied at the application layer (caching prior results and reusing them for repeated identical inputs) rather than assumed from the model call alone.

Beyond the temperature parameter itself, sampling considerations interact with everything else in this module. A well-constrained prompt with explicit success criteria and schema-enforced output will show much less batch-to-batch variance at a *given* temperature than a loosely specified one, because temperature controls variance in word-level token choices, while prompt clarity controls variance in the higher-level decisions being made — which fields to fill in, how to resolve ambiguous cases, what level of detail to include. Both levers matter, and treating temperature as the only tool for consistency, while leaving the prompt itself ambiguous, will under-deliver on reliability no matter how low you set it.

## Detecting Drift and Degradation

The hardest problem in batch processing isn't catching a single obviously bad output — that's usually visible on manual spot-checking or basic validation. It's catching **drift**: a batch run that starts out fine and gradually degrades, or that performs well on the input distribution it was tested against but silently underperforms on a slice of production inputs that differs from that test distribution in some way nobody anticipated. Drift is dangerous specifically because it's silent — there's no error message, no crash, no obvious signal — the outputs just get quietly worse, or wrong in a specific, correlated way, while the pipeline keeps running and reporting success.

A few concrete patterns account for most real-world batch drift:

**Distribution shift within the batch.** If a batch's input characteristics change partway through — say, a batch of support tickets that starts with routine, well-formed English tickets and later includes a cluster of tickets in a different language, or with unusually terse phrasing, or from a newly launched product line the prompt's examples never anticipated — a prompt tuned against the earlier, more typical inputs may quietly perform worse on the later ones without any explicit failure signal. This is the batch-processing analog of the few-shot lecture's warning about examples that don't cover the real range of inputs: if your test set for validating the prompt doesn't span the same range the production batch will, you won't catch this until it's already happened.

**Compounding errors across sequential steps.** In batch pipelines where each item goes through multiple prompt-driven steps, a small error rate at each individual step compounds multiplicatively across the pipeline — a 2% failure rate per step across five sequential steps yields a meaningfully higher end-to-end failure rate than 2%, and if those steps aren't independently monitored, the aggregate degradation can be invisible at any single step while still being substantial by the time it reaches the end of the pipeline.

**Silent format or convention drift.** A model's behavior on a fixed prompt can vary subtly across underlying model versions or over long batch runs in ways that don't trigger any error but do shift output conventions — slightly different phrasing choices, a slightly different way of handling a specific edge case — degrading downstream consistency without any single output looking obviously wrong in isolation.

| Drift Pattern | What Happens | Why It's Dangerous |
|---|---|---|
| Distribution shift within the batch | Input characteristics change partway through | A prompt tuned on earlier inputs quietly underperforms on later ones |
| Compounding errors across sequential steps | Small per-step error rates multiply across a pipeline | Aggregate degradation can be invisible at any single step |
| Silent format or convention drift | Output conventions shift subtly over time or across model versions | No single output looks obviously wrong in isolation |

Detecting drift requires monitoring designed specifically to catch problems that don't announce themselves as errors. Useful techniques include: sampling a consistent percentage of batch outputs for ongoing spot review rather than reviewing only at the start of a batch run; tracking structured metrics across the batch (distribution of output lengths, frequency of specific field values, rate of a rubric-style self-check failing) and watching for statistical shifts over the course of a run rather than just checking pass/fail on individual items; and, where feasible, periodically re-running a fixed, known-good "canary" input through the same batch pipeline and confirming the output still matches the expected pattern, which can catch drift introduced by anything upstream of the prompt itself — a changed system prompt, a changed model version, a changed piece of injected context — not just drift in the input data.

## Batch Processing and the Rest of the Module

Batch processing is best understood as the stage where every other technique in this module gets stress-tested at once. A prompt with vague success criteria that seemed to work fine on the one example you tried by hand will show its ambiguity as inconsistency once run across a thousand varied inputs. Few-shot examples that quietly taught the wrong lesson through an incidental correlation will produce a batch with a systematic, correlated error pattern rather than isolated mistakes — and that pattern is often what first tips you off to the flawed example set in the first place. Structured-output enforcement that relies on prompt wording alone rather than schema constraints will show its failure rate clearly once you're parsing five hundred outputs programmatically instead of eyeballing one. And a self-check or rubric that adds real value on a single output becomes an even more important circuit breaker at batch scale, because it's the mechanism that can flag the specific items worth a closer look rather than requiring a human to review the entire batch by hand.

## A Worked Example

Consider a batch job that classifies ten thousand incoming support tickets by category each night, feeding a routing system that assigns them to the right team the following morning. The prompt was validated against a sample of five hundred historical tickets and performed well — over 97% agreement with human-labeled categories on that sample. Two weeks after launch, the routing team notices a cluster of misrouted tickets, all related to a product feature that launched after the validation sample was collected.

This is textbook distribution shift: the validation sample, however large, was drawn from a period before the new feature existed, so the prompt (and any few-shot examples it relied on) never saw tickets describing it, and had no way to classify a pattern it had never been shown. The batch job didn't announce a failure — every night's run completed successfully and produced output for all ten thousand tickets — the degradation was entirely silent, visible only as a downstream symptom the routing team happened to notice.

The fix illustrates why ongoing monitoring, not just one-time validation, matters. First, the team adds the new feature's tickets to the few-shot example set and to the standing validation batch, so future prompt revisions are checked against the current distribution, not the one from launch day. Second, they add a lightweight statistical check to the nightly run itself: tracking the distribution of assigned categories over time and flagging a run where the distribution shifts more than a set threshold from the recent baseline, which would have caught this specific problem within a day or two rather than two weeks, since a sudden cluster of tickets landing in an unusual category (or in a low-confidence catch-all category) is exactly the kind of statistical signature distribution shift tends to leave. Neither fix is exotic — both are direct applications of the idea that batch validation has to track the real, current input distribution, not a snapshot frozen at prompt-launch time.

**Why it matters for the exam:** Expect questions describing a batch run that degrades partway through, or that performs well on average but poorly on a specific subset of inputs, and asking you to identify the likely cause — usually some form of distribution mismatch between what the prompt was validated against and what the batch actually contained. Also expect questions on appropriate temperature settings for batch tasks (low, for consistency, as the default) and on why spot-checking only the start of a batch run is an insufficient monitoring strategy for catching drift.

<!-- prevnext:start -->

---

| [&larr; Previous: "Traditional" Sanitization](09-traditional-sanitization) | [Next: The Docker Containers of Agentic Workflow Steps &rarr;](11-the-docker-containers-of-agentic-workflow-steps) |
|:---|---:|

<!-- prevnext:end -->
