---
title: Credit Agreements - Observability and Costs
parent: "Extra Module: Solution Architecture Patterns - Financial Documents"
nav_order: 5
duration: "11:21"
---

# Credit Agreements - Observability and Costs
{: .no_toc }

*Video duration: 11:21*

A credit agreement system that extracts terms, flags deviations, and answers questions is only as valuable as the confidence a lending or legal team can place in it — and confidence, for a system that touches legal risk, has to be earned through measurement rather than asserted. This lecture covers three practical concerns that decide whether such a system survives contact with a real lending desk: how you track accuracy against the human review it's meant to support, how you manage the cost of processing genuinely long documents, and how you avoid re-paying that cost every time someone asks a follow-up question.

## Accuracy tracking against human legal review

The right mental model for this system is not "replace legal review" but "make legal review faster and catch what a first pass might miss" — which means the system's accuracy has to be measured *against* human review, continuously, not validated once at launch and left alone. In practice this means every extraction the system produces should, at least initially and then on an ongoing sampled basis, be compared against the same extraction performed independently by a qualified reviewer, with disagreements logged and categorized rather than just tallied as a pass/fail rate.

Categorization matters because not all disagreements carry the same weight. A missed optional field is a very different kind of error from a wrong number on a leverage covenant threshold, and treating them identically in an aggregate accuracy score hides the errors that actually matter. A useful breakdown separates:

- **Extraction errors** — the model pulled the wrong value for a field that's clearly present (a real threshold exists, and the extracted number is different from it). These are the highest-severity errors, because they produce a wrong answer with the appearance of confidence.
- **Omission errors** — the model failed to identify or extract a field that should have been captured, which the checklist-driven review from the workflow lecture is specifically designed to catch, but which still needs to be tracked as a system failure rate, not just silently patched by the human reviewer every time.
- **Interpretation disagreements** — cases where the model's read of an ambiguous clause differs from a reviewer's, not because either is factually wrong but because the clause genuinely admits more than one reading. These are worth tracking separately because they're not really "errors" to fix so much as signals about where the system should flag low confidence and defer, rather than assert a single interpretation.
- **False positives on deviation flagging** — the system flags a clause as non-standard when a reviewer judges it to be within normal variation. High false-positive rates here don't cause legal risk directly, but they erode trust and cause reviewers to start ignoring flags altogether, which is its own failure mode worth measuring.

| Error category | What it means | Severity |
|---|---|---|
| Extraction errors | Wrong value pulled for a field that is clearly present | Highest — a wrong answer with the appearance of confidence |
| Omission errors | A field that should have been captured was missed | High — should be caught by checklist review, still tracked |
| Interpretation disagreements | Model's read of an ambiguous clause differs from a reviewer's | Signal to flag low confidence, not necessarily an error |
| False positives (deviation flagging) | Clause flagged as non-standard when it's normal variation | Low legal risk, but erodes trust in the flags over time |

Tracking these categories over time, by deal type and by clause type, does two things: it tells you where the system is reliable enough to lighten the human review burden (say, routine facility-amount and maturity-date extraction, which tend to be unambiguous and low-error) and where it categorically isn't yet (say, complex basket-stacking provisions in negative covenants, which are notoriously hard even for experienced attorneys to parse correctly). That per-category reliability picture is what should actually drive how much weight a lending team puts on the system's output for a given field — **not a single blended accuracy number that averages away exactly the distinction that matters**.

{: .important }
> Never report accuracy as one blended number. A wrong leverage-covenant threshold and a missed optional field are not the same failure — averaging them into a single score hides the highest-severity error category (confident-looking wrong values) behind the lowest-severity one.

## The cost of processing long documents

Credit agreements run long, and every architectural choice from the earlier lectures in this module has direct cost consequences worth being explicit about. A naive approach — feeding the full document into context for every task — multiplies cost unnecessarily, because most tasks only need a fraction of the document (recall the chunking-by-legal-structure approach from the context architecture lecture). The cost-efficient pattern processes a document once at ingestion time — classifying deal type, chunking by article, extracting and indexing defined terms — and then reuses that processed representation across every downstream task, rather than re-reading and re-parsing the raw document from scratch each time a new question comes in.

**Prompt caching** is the single most impactful lever available here, and it maps directly onto the structure this module has already established. The Definitions article and the full text of the agreement under review are stable within the scope of working on a given deal — they don't change between the extraction pass, the deviation check, and any follow-up Q&A a reviewer runs later that day. Structuring the system so that this stable material sits in a cached prefix, with only the task-specific instruction and any newly retrieved precedent clauses varying between calls, means the bulk of the token cost for a long document is paid once rather than on every single call that touches it. For a two-hundred-page agreement fielding a dozen different extraction and Q&A calls over the course of a deal review, the difference between caching the document once and re-sending it uncached on every call is not a marginal optimization — it's frequently the difference between a system that's economically viable at scale and one that isn't.

Beyond caching, model selection by task tier is worth being deliberate about rather than defaulting to the largest available model everywhere. Deal-type classification and straightforward field extraction — well-defined, narrow, high-volume tasks — are strong candidates for a smaller, faster, cheaper model, since the task doesn't require the deepest reasoning capability available. Nuanced deviation assessment and answering genuinely ambiguous legal questions warrant the most capable model the system has access to, because that's where reasoning quality has the largest effect on whether the output is trustworthy. Routing tasks to the right tier, rather than running every call at the same (typically maximum) capability level, is a direct cost lever with essentially no accuracy downside if the tiering is done correctly.

## Caching strategies for repeated queries against the same agreement

Beyond prompt caching at the token level, there's a second, higher-level caching opportunity specific to how these documents actually get used: the same agreement gets queried repeatedly, by multiple people, over the life of a deal. A credit analyst extracts terms for the deal database on day one; a portfolio manager asks about covenant headroom before a compliance certificate is due three months later; outside counsel asks about MFN implications during a subsequent add-on facility a year after that. Each of these interactions, especially the first extraction pass, produces artifacts — the deal-type classification, the extracted field set, the chunked and indexed document, the resolved cross-references — that don't need to be regenerated from scratch for every subsequent query against the same agreement.

The practical design is a **document-level result cache**, keyed to the specific agreement (and its version, since amendments and side letters do change the operative terms over time), that stores the already-extracted structured data and the already-processed, chunked, cross-reference-resolved document representation. A new question against an agreement that's already been processed hits this cache first: routine questions ("what's the current leverage covenant threshold") may be answerable directly from the cached extraction without any new model call at all, while more open-ended questions still invoke the model but do so against the already-chunked, already-indexed document rather than repeating the ingestion work. This matters more for credit agreements than for many other document types precisely because of the query pattern — a small number of long-lived agreements, each queried many times over months or years, rather than a large number of documents each queried once. Getting this caching layer right is often what separates a system that's genuinely cheap to run in steady state from one whose costs scale linearly with every question anyone ever asks, which for an actively managed loan portfolio can be a very large number of questions over a deal's life.

**Architecture Notes:** Accuracy tracking against human review and category-specific error rates are Context-layer concerns as much as measurement concerns — they tell you where the model's context (or its interpretation of ambiguous context) is insufficient and a human checkpoint is non-negotiable. Prompt caching and document-level result caching are Workflow-layer optimizations that exploit the fact that the pipeline from Lecture 2 is fixed and repeatable: because the same document gets processed the same way across many calls, its stable material is a natural candidate for reuse rather than regeneration.

<!-- prevnext:start -->

---

| [&larr; Previous: Credit Agreements - RAG & Retrieval Architecture](04-credit-agreements-rag-and-retrieval-architecture) | [Next: LPA/Subscription Agreements - Overview &rarr;](06-lpa-and-subscription-agreements-overview) |
|:---|---:|

<!-- prevnext:end -->
