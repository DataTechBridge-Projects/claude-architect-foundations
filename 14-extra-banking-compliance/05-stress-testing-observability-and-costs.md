---
title: Stress Testing - Observability and Costs
parent: Extra Module: Solution Architecture Patterns - Banking Compliance
nav_order: 5
duration: "9:35"
---

# Stress Testing - Observability and Costs
{: .no_toc }

*Video duration: 9:35*

A stress-testing system's job doesn't end when a narrative gets approved and submitted. In a very real sense, the system's most important output is the record it leaves behind — because a regulatory examiner can, at any point in the following months or years, ask the bank to reconstruct exactly how a specific number in the submission was produced. Observability in this domain is built primarily to answer that question, and only secondarily to answer the more familiar "is the system healthy" questions an engineering team would normally ask of production software.

## The Audit Trail Regulators Actually Expect

A supervisory examination of a stress-test submission doesn't accept "the model said so" as an explanation for anything. What it expects is a reconstructable chain: which scenario document (with its verified cycle tag) fed which model run, against which portfolio snapshot, producing which output, reviewed and approved by which named individual, on what date. Every one of those links has to be logged at the time it happens, not reconstructed after the fact from memory or from scattered emails. For a Claude-based assistant sitting inside this pipeline, that means every generation — scenario extraction, orchestration step, narrative draft — is logged with its full input context: which retrieved documents were included, which portfolio snapshot identifier was referenced, what the model produced, and which human reviewed and either approved or revised it before it moved forward.

This logging has to capture not just the final approved version of each artifact but the intermediate ones too, including anything a reviewer rejected or sent back for revision. A rejected draft and the reason it was rejected are themselves valuable audit evidence — they demonstrate that the review gate is a real control rather than a rubber stamp, which is precisely the kind of evidence a supervisor examining the bank's control environment (as distinct from just the numbers) is looking for. A system that only retains final, approved outputs and silently discards everything upstream of them has quietly destroyed the evidence that its own review process actually functions.

## Reproducibility as a First-Class Requirement

Because a run might be scrutinized long after the cycle it belongs to has closed, the system has to support exact reproduction of any given run, not just an approximate account of what happened. This is where the versioning discipline from the context-architecture lecture pays off directly: because scenario documents, methodology documents, and portfolio snapshots are all immutably versioned and tagged, a logged run identifier is enough to pull back the *exact* set of inputs that produced a given output, months or years later. Reproducibility also has a narrower, more immediate use during the cycle itself: when a quant reviewer sends a draft back with a change, being able to re-run the same stage with a modified input and confirm the change produced the expected downstream effect is what keeps iteration fast without sacrificing traceability.

It's worth being explicit that "reproducible" here means bit-for-bit traceable to a specific set of versioned inputs — not that re-running the exact same prompt against the model is guaranteed to produce identical prose. Narrative drafting is expected to vary in wording between runs; what has to be reproducible is the *evidentiary chain*, not the literal text. The audit standard is "can we show exactly what this was based on," not "can we regenerate the identical sentence."

## Cost Architecture for a Periodic, Not Continuous, Workload

Stress testing has an unusual cost profile compared to most Claude-based production systems, and it's worth designing for deliberately rather than defaulting to whatever infrastructure pattern a continuously running system would use. The workload is intensely concentrated: for a few weeks around each regulatory cycle, the system runs hard across dozens of portfolio segments, multiple scenarios, and repeated rounds of drafting and revision. Outside that window, usage drops close to zero — there's no ongoing customer-facing traffic to serve, no steady drip of requests that justifies always-on capacity sized for peak load.

That profile argues against provisioning as if this were a continuously loaded system and toward architecture that treats the annual cycle as the unit of capacity planning. A few concrete implications follow:

- **Batch-oriented processing** during the active window, rather than always-on low-latency serving, since the internal users of this system are quant analysts working through a structured task list, not end customers expecting instant responses.
- **Prompt caching** applied aggressively to the large, stable inputs that get reused across many runs within a cycle — the scenario documents and methodology guidance don't change hour to hour once a cycle's inputs are locked, so the same context can be cached across the many narrative-drafting calls that reference it, meaningfully cutting the per-call cost of a heavy-context task.
- **Right-sized model selection by stage** — a structured extraction task (pulling assumption values out of a scenario document into a schema) is a different cost-and-capability profile than open-ended narrative synthesis, and treating every stage as needing the same model tier wastes budget on the simpler stages.
- **Cost tracking scoped to the cycle**, not averaged over the year, since a monthly cost dashboard that smooths a concentrated few weeks of heavy usage into an annual average will misrepresent what the system actually costs to run and make it hard to budget accurately for the next cycle.

## Monitoring What Actually Matters Here

Traditional uptime and latency monitoring still applies, but it's not the center of gravity for this system the way it would be for a customer-facing product. What matters more is monitoring the *quality signals* the review process generates: how often reviewers send drafts back for revision, what categories of error recur (a stale citation, a misapplied scenario assumption, a narrative claim not supported by the retrieved context), and whether those rates improve or degrade across cycles as the underlying prompts, retrieval filters, and methodology documents evolve. That data feeds back into refining the system for next cycle — and, not incidentally, is itself part of the evidence base a bank can show a supervisor asking how it governs and improves the tools it uses to produce regulatory submissions.

## Architecture Notes

This lecture shows the **Context** layer's provenance discipline paying dividends downstream: because every input is versioned and tagged, the audit trail regulators demand is a byproduct of good context architecture rather than a separate system bolted on afterward. The cost model, meanwhile, is a direct consequence of the **Agentic** layer's low-autonomy, staged design combined with a genuinely periodic workload — caching and batch processing fit a system that runs hard for weeks and idles the rest of the year, which is a very different cost shape than a continuously served agentic product.
