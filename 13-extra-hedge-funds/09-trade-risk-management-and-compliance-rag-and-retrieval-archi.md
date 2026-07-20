---
title: Trade Risk Management/Compliance - RAG & Retrieval Architecture
parent: Extra Module: Solution Architecture Patterns - Hedge Funds
nav_order: 9
duration: "9:08"
---

# Trade Risk Management/Compliance - RAG & Retrieval Architecture
{: .no_toc }

*Video duration: 9:08*

Retrieval in the compliance domain has a requirement that goes beyond anything in the idea generation use case: it's not enough for the system to find the *right* rule, it has to be able to prove, after the fact, exactly which version of that rule it applied and when that version was in effect. Regulations get amended, internal policies get revised, restricted lists change daily — and a retrieval system that can't answer "which version of this rule governed the review on this date" has built a compliance tool with a hole in the middle of it, no matter how accurate its retrieval feels in the moment.

## What's Indexed

The corpus here is narrower than the idea generation use case's four-way split, but each category demands more rigor. **Regulatory text** — the actual rules issued by the relevant regulatory bodies for the fund's jurisdictions and strategies — is indexed at a fine granularity, typically by rule or sub-rule rather than by whole regulation, so retrieval can surface the specific provision that applies rather than an entire lengthy document a reviewer then has to search manually. Each indexed unit carries its citation (the formal rule number or section), its effective date, and — critically — its relationship to prior versions, so the system can trace a rule's history rather than treating each amendment as an unrelated new document.

**Internal compliance policies** — the fund's own interpretation and implementation of external regulatory requirements, often stricter than the regulatory minimum — are indexed the same way, by policy section, with the same versioning discipline. These need to be explicitly linked to the regulatory text they implement, so that when the agent surfaces a policy, it can also surface the underlying rule that policy exists to satisfy, which matters when the two say different things and a reviewer needs to understand why.

**Restricted and watch lists** are the most dynamic corpus in the system — these change daily or even intraday, and treating them like static indexed documents refreshed on a batch schedule is a real risk. These are best handled less like a RAG corpus and more like a live lookup: a direct query against the current list rather than a retrieval over a periodically re-embedded index, precisely because the cost of retrieving a restricted-list snapshot that's a day stale is a trade clearing against a name that was added to the list that morning.

**Counterparty and jurisdictional reference data** — standard documentation about which rules apply to which counterparty types, entity structures, and cross-border scenarios — rounds out the corpus. This changes less often than restricted lists but still needs a defined refresh cadence, since new counterparty relationships and new jurisdictional guidance both need to enter the index promptly.

## Retrieval Strategy

Retrieval for this system prioritizes precision and provenance over the more exploratory, breadth-oriented retrieval used in idea generation. Where the idea generation system's screening agent benefits from a generous, high-recall first pass, the compliance review agent's retrieval should favor high precision on the specific rules that plausibly apply to the trade's instrument type, jurisdiction, and counterparty — a narrower, more targeted query built from structured trade attributes (instrument class, jurisdiction, counterparty type) rather than a broad semantic search over the whole regulatory corpus. This is a case where combining structured filtering with semantic search meaningfully outperforms semantic search alone: filter the corpus down to rules tagged for the relevant jurisdiction and instrument class first, and only then run semantic retrieval within that filtered set to find the specific provisions relevant to the trade's particular structure.

A second retrieval pass, structurally similar to the idea generation system's asymmetric vetting queries, is worth running here too: after identifying the rules that plausibly support clearing the trade, the agent should also explicitly query for rules that might *prohibit* or restrict it — a targeted search for exclusions, conditions, and exceptions rather than relying on the same query to surface both the general rule and its carve-outs. Compliance text is notorious for burying the operative constraint in an exception clause several paragraphs after the general rule, and a retrieval strategy that doesn't specifically hunt for exceptions will systematically under-surface them.

## Versioning: Citing the Rule Version That Was Actually Applied

This is the requirement that most sharply distinguishes this use case from anything else in the course, and it deserves to be treated as a first-class architectural requirement rather than a nice-to-have metadata field. Every rule in the index is stored with an explicit version identifier and an effective date range (when it became effective, and when — if applicable — it was superseded). When the retrieval system serves a rule to the compliance review agent, it serves the version that was effective *as of the trade's review date*, not simply the most recently indexed version, which matters enormously for any retrospective review: if a compliance officer or a regulator asks six months later why a particular trade cleared, the system needs to be able to reconstruct exactly which rule text was in force at that moment, not the rule text as it reads today.

In practice, this means the index is never allowed to simply overwrite an old rule version with a new one — amendments are stored as new versions linked to their predecessor, with the predecessor's effective date range closed out rather than deleted. The agent's output is required to cite not just "Rule 15c3-1" or the internal policy name, but the specific version and effective date it retrieved and reasoned over, embedded directly in its structured assessment. This citation isn't cosmetic — it's the mechanism that makes the whole review defensible after the fact, and it's checked as part of the system's own quality gates: an assessment that references a rule without a version and effective date is treated as incomplete and rejected before it reaches a human reviewer, the same way a malformed tool call would be rejected before reaching a model in any other system.

## Freshness for the Fastest-Moving Inputs

Restricted lists and counterparty status data need freshness guarantees stronger than anything in the idea generation use case, because the cost of staleness here is a trade clearing against a name it shouldn't have. Rather than relying on periodic re-indexing, these are best served through a direct, real-time lookup path that bypasses the semantic retrieval index entirely for exact-match checks — a proposed trade's counterparty and instrument names are checked against the live current list before the agent's broader rule-based review even begins, so a restricted-list hit is caught by a fast, deterministic check rather than depending on the retrieval system's semantic search having recently re-indexed the list. This mirrors the earlier point about deterministic versus agentic tiers: exact-match restricted-list checking belongs in the same deterministic tier as position-limit checking, not in the retrieval-and-reasoning path, precisely because it's too consequential and too mechanical to trust to embedding similarity and refresh timing.

## Architecture Notes

Rule versioning with explicit effective-date ranges is this use case's sharpest expression of the Context layer's provenance principle — provenance here isn't a nice-to-have for a citation footnote, it's the difference between a defensible compliance decision and one that can't be reconstructed under audit. Routing restricted-list checks to a deterministic live lookup rather than the semantic retrieval path reinforces the Agentic layer's tiering decision from the earlier lecture: the fastest-moving, highest-consequence data belongs in the mechanical tier, not wherever retrieval happens to be most convenient to wire up.
