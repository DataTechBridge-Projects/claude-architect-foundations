---
title: Regulatory Capital Validation - Observability and Costs
parent: "Extra Module: Solution Architecture Patterns - Banking Compliance"
nav_order: 10
duration: "9:37"
---

# Regulatory Capital Validation - Observability and Costs
{: .no_toc }

*Video duration: 9:37*

A validation system's entire reason for existing is to be trusted enough that a human validator can rely on its flags — and enough that a supervisor can rely on the bank's account of how discrepancies get caught and resolved. That trust has to be earned continuously, not established once at launch, which is why observability for this use case centers on two things a generative system like the stress-testing assistant doesn't have to worry about nearly as much: a complete resolution record for every flag, and ongoing measurement of the system's own accuracy.

## Logging Every Flag and Its Resolution

Every discrepancy the system surfaces has to be logged with the full evidentiary chain behind it: which calculation step was checked, which regulatory provision and (where relevant) which internal interpretation it was checked against, what the calculation actually did, and the specific nature of the divergence identified. That's the input side of the log. The output side — arguably the more important half for audit purposes — is what happened next: which human validator reviewed the flag, what they determined (confirmed as a genuine discrepancy, or determined to be a false positive and why), and, for confirmed discrepancies, what remediation was taken and when.

This resolution record matters for two audiences at once. Internally, it's what lets the bank demonstrate that its control environment actually functions — that flagged issues get tracked to closure rather than accumulating in an unreviewed backlog. Externally, it's exactly the kind of evidence a supervisor examining the bank's model risk management or internal controls will ask for: not just "does the bank have a validation process" but "show me the last dozen discrepancies this process found and how each one was resolved." A system that logs flags but not their resolutions has built half of what this domain actually requires.

It's worth logging clean-pass verdicts too, at least the sampled subset described in the agentic-architecture lecture, along with the outcome of that sampled review. A supervisor asking "how do you know this tool isn't missing things" deserves a better answer than "we only kept records of what it caught" — the sampled clean-pass audit trail is direct evidence that the system's negative results are being checked, not just trusted.

## Tracking False Positives and False Negatives

Because this is fundamentally a verification system, its most important ongoing health metric isn't uptime or latency — it's the rate at which its verdicts turn out to be wrong in either direction, and those two directions have to be tracked separately because they have very different costs and very different visibility.

A **false positive** — flagging a discrepancy that a human reviewer determines isn't actually one — is costly in reviewer time and, if it happens often enough, corrosive to trust in the system; validators who've been burned by noisy flags start rubber-stamping them, which quietly defeats the purpose of having a human review step at all. False positives are relatively easy to measure, because every flag gets a human resolution that explicitly records whether it was confirmed or dismissed, so the false-positive rate falls directly out of the resolution log described above.

A **false negative** — a real discrepancy the system missed entirely — is far more dangerous and far harder to measure, precisely because by definition the system didn't flag it, so there's no natural record pointing an observer toward the miss. This is why the sampled review of clean passes isn't a nice-to-have; it's the primary mechanism for estimating the false-negative rate at all. Some false negatives will also surface later through other channels — a subsequent regulatory examination, a later validation cycle using updated methodology, or an unrelated audit — and every one of those has to be fed back into the system's tracked accuracy record and, ideally, into a root-cause review of why the check missed it, since a recurring blind spot in how calculations get decomposed into checkable steps is a system design problem, not a one-off miss.

| | False Positive | False Negative |
|---|---|---|
| Definition | Flags a discrepancy that isn't one | Misses a real discrepancy |
| Ease of measurement | Easy — every flag gets a human resolution | Hard — no natural record points to the miss |
| Primary cost | Reviewer time; erodes trust, risks rubber-stamping | Bank operates on a miscalculated capital position |
| Main detection mechanism | Resolution log | Sampled review of clean-pass verdicts |

Both rates should be tracked over time and segmented — by calculation type, by which methodology area is involved, by which model or business line the calculation belongs to — because an aggregate accuracy number can hide a system that's excellent at some categories of check and unreliable at others. A validation tool that's 98% accurate in aggregate but consistently misses discrepancies in one specific, high-risk calculation category has a serious problem that an aggregate metric will actively conceal.

## Cost Tradeoffs

Regulatory capital validation doesn't share stress testing's sharply seasonal load — capital ratios get calculated and validated on a recurring reporting cycle, often quarterly, which makes the workload steadier than the annual stress-test crunch but still far from continuous, always-on traffic. That shapes the cost architecture differently from either a seasonal spike or a genuinely continuous production system.

A few cost considerations are specific to this use case. First, the per-check cost has to account for retrieving and reasoning over *two* source families for every provision with a documented interpretation — regulatory text and internal interpretation — which is inherently more retrieval and comparison work than a single-source check, and that added cost is the direct price of the three-way verdict structure the previous lecture described; it's not overhead to be optimized away; it's what the domain requires to avoid the false-positive flood or false-negative blind spot a single-source design would produce. Second, the sampled clean-pass review adds cost precisely where it produces no visible finding most of the time — a lever that's tempting to cut when budgets tighten, since it looks like paying for nothing happening, but that's exactly the wrong place to cut given that it's the main defense against undetected false negatives. Third, the calculation-decomposition step — breaking a calculation into individually checkable pieces — is worth caching and reusing across validation cycles for calculation types whose structure doesn't change quarter to quarter, since re-deriving the same decomposition from scratch each cycle is pure waste when only the underlying numbers and applicable methodology version have changed.

Segmenting cost tracking by calculation type and methodology area, the same way accuracy is segmented, also lets the bank make an informed decision about where to invest more validation effort — a high-risk calculation category can justify a more expensive, more thorough check (larger context, more sampled clean-pass review) than a low-risk, stable one, and that's a defensible, documentable resource-allocation decision rather than a uniform cost applied everywhere regardless of actual risk.

## Architecture Notes

This lecture is where the **Agentic** layer's verification-not-generation framing and the **Context** layer's provenance discipline both cash out as measurable outcomes: false positives and false negatives are the concrete, trackable expression of "is this checking agent actually doing its job," and they can only be measured because the resolution log and sampled clean-pass review were built into the **Workflow** layer from the start. The cost model follows the same logic as stress testing's — spend where the domain's specific risks require it, track it at a granularity that matches how risk is actually distributed — but arrives at a different shape because the workload here is steadier and the two-source comparison is intrinsically more expensive per check than a single generative draft.

<!-- prevnext:start -->

---

| [&larr; Previous: Regulatory Capital Validation - RAG & Retrieval Architecture](09-regulatory-capital-validation-rag-and-retrieval-architecture) | [Next: Extra Module: Solution Architecture Patterns - Financial Documents &rarr;](../15-extra-financial-documents/) |
|:---|---:|

<!-- prevnext:end -->
