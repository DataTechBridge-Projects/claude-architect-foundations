---
title: Stress Testing - RAG & Retrieval Architecture
parent: Extra Module: Solution Architecture Patterns - Banking Compliance
nav_order: 4
duration: "7:56"
---

# Stress Testing - RAG & Retrieval Architecture
{: .no_toc }

*Video duration: 7:56*

The previous lecture established that context has to carry the *correct cycle's* scenario and portfolio data. This lecture covers the retrieval layer that's actually responsible for making that true — the system that decides, out of a growing multi-year corpus of regulatory guidance and internal methodology documents, exactly which passages Claude sees when it's assembling a scenario or drafting a narrative for the current cycle.

## What's Being Retrieved

Two broad document families sit behind this system, and they behave very differently as retrieval targets. **Regulatory guidance** includes the regulator's published scenario documents, instructional guidance on how banks should apply stress-test methodology, and any supervisory feedback letters from prior cycles that speak to how the bank's own submissions have been received. This corpus is externally authored, updated on the regulator's schedule (typically annually, sometimes with mid-cycle clarifications), and authoritative in the strict sense — when regulatory guidance says something, that's the rule, full stop. **Internal methodology documents** are the bank's own model documentation, its interpretation of how to apply regulatory guidance to its specific portfolio structure, and its own past submission narratives. This corpus changes more frequently, as models get revalidated and methodology gets refined, and it's authoritative about *how the bank does things* rather than about what the rule says.

Retrieval has to serve both corpora without conflating them, because a narrative-drafting task typically needs both at once: what does the regulator's guidance require, and what is the bank's documented, approved way of meeting that requirement. Blending them without clear attribution in the retrieved context risks the model treating an internal methodology choice as if it were a hard regulatory requirement, or vice versa — a distinction that matters enormously if a supervisor later asks the bank to justify a specific choice.

## The Stale-Cycle Problem

The central retrieval challenge in this domain, previewed in the context lecture, is preventing the system from surfacing correct-looking guidance from the wrong cycle. A naive retrieval setup — embed everything, retrieve by semantic similarity to the query — will happily return last year's scenario instructions alongside this year's, because they're about the same topic, phrased similarly, and genuinely relevant to *understanding* the current methodology even though they must not be used to *populate* the current submission. A retrieval system built for this domain has to treat cycle-currency as a hard filter, not a ranking signal. In practice this means:

- Every document in the corpus carries a mandatory, validated **cycle metadata tag** — the regulatory cycle year and, where applicable, the specific guidance version — assigned at ingestion, not inferred later from content.
- Retrieval queries tied to "assemble this cycle's scenario" or "draft this cycle's narrative" apply a **hard filter on the current cycle tag** before semantic ranking runs at all, so an off-cycle document is excluded from the candidate set entirely rather than merely ranked lower.
- Prior-cycle documents remain retrievable, but only through an explicitly different query path — the "show me what changed since last cycle" or "how did we explain this last time" use case — where they're labeled unambiguously as historical in the retrieved context, exactly as described in the context-architecture lecture.

This is a case where getting the retrieval *filter* right matters more than getting the retrieval *ranking* right. A system that ranks the correct document first 95% of the time but has no hard filter will, in the remaining cases, silently hand the model a stale rule dressed up as current — and because the wording of consecutive years' guidance is often structurally similar, that failure is exactly the kind that's hard to catch by inspection.

## Retrieval Over Internal Methodology

Internal methodology retrieval carries a related but distinct challenge: methodology documents reference specific models, and models get revalidated on their own schedule, independent of the regulatory cycle. A methodology document describing "the current approach" to a given loss-forecasting model has to be tied to that model's validation status, not just its publication date, so the system doesn't retrieve a methodology description for a model version that's since been superseded. This is handled the same way as cycle tagging — a validated status tag at ingestion, enforced as a filter — but it's worth calling out separately because it's a second, independent axis of staleness beyond the regulatory-cycle one, and a retrieval system that only guards against one of the two will still produce grounded-looking errors from the other.

## Grounding the Narrative in Citable Sources

The payoff of getting retrieval right shows up most visibly in the narrative-drafting stage described in the agentic-architecture lecture. Every sentence of narrative that makes a claim about scenario assumptions or methodology should be traceable to a specific retrieved passage, with that passage's source and cycle tag available for a reviewer to check inline rather than needing to go hunting for it separately. This isn't just good practice — it's what makes the mandatory review gate fast enough to be practical at submission scale. A reviewer checking "does this sentence accurately reflect current guidance" against an attached citation is a much faster task than re-deriving the answer from scratch, and it's the retrieval layer's citation discipline that makes that shortcut trustworthy rather than a shortcut around actual verification.

## Architecture Notes

This lecture is the **Context** layer's data-provenance principle implemented as retrieval infrastructure: hard filters on cycle and validation-status metadata prevent an off-cycle or superseded document from ever entering the candidate set, rather than relying on semantic ranking to sort it out. The citation discipline that grounds every narrative claim also serves the **Prompt** layer, since a drafting prompt that requires inline sourcing produces output a human can verify quickly — which is precisely what keeps the **Workflow** layer's review gates fast enough to survive a real regulatory deadline.
