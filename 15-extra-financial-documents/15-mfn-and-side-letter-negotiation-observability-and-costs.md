---
title: MFN & Side Letter Negotiation - Observability and Costs
parent: "Extra Module: Solution Architecture Patterns - Financial Documents"
nav_order: 15
duration: "11:01"
---

# MFN & Side Letter Negotiation - Observability and Costs
{: .no_toc }

*Read duration: 11:01*

Every use case in this module has had its own risk profile, but MFN and side letter analysis has the sharpest one: the failure mode isn't a wrong extracted number or a template mismatch, it's a legal obligation the fund doesn't know it has, discovered — if it's discovered at all — by an investor's counsel rather than the fund's own team. That asymmetry between quiet, hard-to-detect errors and expensive, hard-to-undo consequences should shape observability and cost design more directly here than in either of the earlier two modules.

## The high stakes of being wrong

It's worth being concrete about what "wrong" costs in this domain. A missed MFN trigger — the system fails to flag that a proposed term should have activated another investor's election right — doesn't surface as an obvious error at the time it happens. It surfaces later, potentially much later, when that other investor's counsel independently notices the disparity, at which point the fund is dealing with a dispute, a retroactive election claim, and a credibility problem about the rigor of its own side letter management, rather than a routine correction caught in the ordinary course of business. A false positive — flagging an MFN trigger that a careful reading shows doesn't actually apply — is cheaper but not free: it costs legal review time and, if it happens often enough, teaches reviewers to discount the system's flags, which quietly erodes the very safeguard the system exists to provide. Both directions of error matter, but the cost curve is steeply asymmetric toward missed triggers, and observability design should reflect that asymmetry rather than treating both error types as equally weighted contributors to a single accuracy score.

That asymmetry argues for a specific measurement posture: because false negatives are the expensive, hard-to-detect failure, the system should be validated with disproportionate attention to *recall* — did it surface every side letter that plausibly could be relevant — even at some cost to precision. It's more tolerable for the system to surface five candidate side letters where only two are truly relevant, forcing a reviewer to spend a few extra minutes ruling out the other three, than to surface two when a third genuinely relevant one existed and went unmentioned.

{: .important }
> Optimize this system for recall over precision, and never let overall accuracy be the headline metric. A missed MFN trigger is a dispute discovered by an investor's counsel months later; a false positive is a few extra minutes of legal review — those are not equivalent costs, and a blended score treats them as if they were. Observability metrics that track only overall accuracy, without decomposing into recall on known-relevant cases versus precision on flagged cases, will systematically under-weight exactly the failure mode that matters most here.

| Error type | How it surfaces | Cost |
|---|---|---|
| Missed MFN trigger (false negative) | Often not until an investor's counsel raises it independently | Dispute, retroactive election claim, credibility damage |
| False positive | Immediately, during legal review | Review time; erodes trust in flags if it happens often |

## Building and maintaining a recall benchmark

Because missed triggers are the central risk, and because they're inherently hard to catch by construction (a missed trigger looks, from the outside, identical to "there was nothing to find"), the practical mitigation is to build a **deliberate recall benchmark**: a curated set of past MFN scenarios — real historical cases where a term was proposed and an MFN trigger either did or didn't apply, ideally including some genuinely subtle ones where the trigger depended on a qualifier a careless read would miss — that the retrieval and analysis pipeline is periodically re-run against to confirm it still surfaces what it should. This benchmark should be maintained and expanded over time, specifically incorporating any case where a human reviewer catches something the system missed, so that every near-miss in production becomes a permanent regression test rather than a one-off correction. Without this kind of standing benchmark, recall degradation — say, from a change to the embedding model, a shift in how new side letters get digested, or simply an accumulation of new side letters whose normalized digests weren't reviewed as carefully as the earlier ones — can go undetected indefinitely, precisely because a missed trigger doesn't announce itself.

## The mandatory legal review gate

Given the stakes, this is the one use case in the module where a human review gate isn't a design choice weighed against efficiency — it's a non-negotiable structural requirement, and the architecture should be built to make skipping it difficult rather than merely discouraged. **Every MFN determination the system produces, whether it concludes a trigger applies or concludes it doesn't, routes to fund counsel for review** before any communication to an investor or any update to the fund's official side letter registry. This is stricter than the review gates in the credit agreements and LPA modules, where routine, high-confidence extractions could reasonably flow through with lighter-touch spot-check review — here, even a system conclusion of "no trigger applies" needs review, precisely because a false negative is the failure mode that matters most and a confident-sounding "no" is exactly the kind of output that's easiest for a reviewer to wave through without scrutiny if the gate isn't structured to demand real engagement with the reasoning shown, not just a checkbox approval.

Making that gate effective, rather than a rubber stamp, means the system's output has to give the reviewer something substantive to evaluate — the structured comparison table from the previous lecture, with the specific side letters considered, the reasoning for inclusion or exclusion, and an explicit accounting of the recall strategy used (which categories were searched, which qualifiers were checked) — so the reviewer can assess whether the *search* was thorough, not just whether the *conclusion* looks reasonable given what was searched. A reviewer who only sees the final answer has no way to catch a recall failure; a reviewer who sees the search strategy can.

## Cost of the more retrieval-heavy comparison workload

This use case is more expensive per analysis than the templated modules, and that expense is inherent to the task rather than an optimization failure to be engineered away. Every MFN check potentially touches a meaningful fraction of the fund's side letter population — reading normalized digests across dozens or hundreds of letters, pulling full text for whichever ones the categorical filter surfaces as candidates, and running agentic reasoning across each candidate rather than a single deterministic pass. That's a fundamentally heavier workload than templated field extraction from one subscription document, and cost optimization here has to work with that reality rather than fight it.

The most effective lever is the same digest-layer investment described in the retrieval lecture: the expensive work of normalizing a side letter's bespoke language into a structured, comparable digest happens once, at ingestion, and every subsequent MFN check reuses that digest rather than re-reading and re-interpreting raw clause text from scratch each time. Because a given side letter might be checked against dozens of proposed terms over a fund's life, amortizing the ingestion cost across all of those checks is what keeps steady-state cost manageable — the expensive step is paid once per side letter, not once per query. Model tiering also applies, though less aggressively than in the templated modules: the categorical filtering pass (does this side letter even have an MFN clause covering this term category) is cheap and can run on a lighter-weight model, while the substantive comparability judgment — does this qualifier actually apply, does this proposed term functionally match what's already been granted — genuinely warrants the most capable model available, because that judgment is precisely where the recall-critical reasoning lives and where a cheaper model's shortcuts are most likely to produce exactly the quiet, hard-to-detect miss this whole module is built to prevent.

**Architecture Notes:** The asymmetric cost of false negatives versus false positives should directly shape what gets measured — recall-weighted benchmarks over blended accuracy scores — which is a Context-layer discipline about ensuring the system's working set (and its search strategy) is actually complete, not just about the correctness of any single answer. The mandatory, unskippable legal review gate is the clearest Workflow-layer safeguard in this entire module, deliberately wrapped around the Agentic layer's autonomy precisely because the task's open-endedness is also what makes its failure modes the hardest to catch without a human in the loop.

<!-- prevnext:start -->

---

| [&larr; Previous: MFN & Side Letter Negotiation - RAG & Retrieval Architecture](14-mfn-and-side-letter-negotiation-rag-and-retrieval-architectu) | [Next: MCP Server Building &rarr;](../16-mcp-server-building/) |
|:---|---:|

<!-- prevnext:end -->
