---
title: Regulatory Capital Validation - RAG & Retrieval Architecture
parent: Extra Module: Solution Architecture Patterns - Banking Compliance
nav_order: 9
duration: "8:23"
---

# Regulatory Capital Validation - RAG & Retrieval Architecture
{: .no_toc }

*Video duration: 8:23*

The context lecture established what has to be anchored in front of the model for a single validation check: current methodology, current thresholds, and the calculation under review. This lecture covers how retrieval finds the right passages of methodology to anchor with — and confronts a problem more structurally awkward than the stale-cycle problem in stress testing: here, the two authoritative-seeming sources the system retrieves from can actually *disagree with each other*, and the system has to make that disagreement visible rather than silently resolve it in either direction.

## Two Corpora, One of Them Self-Contradicting

Retrieval in this domain draws on two document families that parallel the stress-testing use case's structure but carry a sharper edge. **Regulatory text** is the actual, authoritative source: the published capital rules, technical standards, and any official regulatory guidance clarifying how provisions should be applied. This is the letter of the law, in the strict sense that when a conflict arises, this is what ultimately governs. **Internal policy interpretations** are the bank's own documented positions on how to apply that regulatory text to its specific business — memos, policy documents, and model-governance decisions written by the bank's own compliance and risk teams, often addressing genuine ambiguity in the regulatory text or documenting a considered judgment call about how a provision applies to a product or structure the regulation doesn't address in explicit detail.

The uncomfortable reality this domain has to be architected around is that internal interpretations sometimes diverge from a strict reading of the regulatory text — not necessarily through error, but often through a defensible judgment call made in good faith about genuinely ambiguous language, sometimes even discussed with the regulator informally and never contradicted, but also never formally codified into the regulation itself. A validation system that retrieves only internal interpretation will validate the bank's calculations against the bank's own possibly-generous reading of the rules — silently baking any drift into a clean bill of health. A system that retrieves only the regulatory text and ignores internal interpretation will flag every calculation that relies on a documented, defensible interpretive judgment as a discrepancy, burying real problems in a flood of noise the validation team has already resolved once and shouldn't have to re-litigate on every check.

## Retrieval That Surfaces Both, Explicitly Separated

The architectural answer is retrieval that deliberately returns both sources for any provision that has a documented internal interpretation, tagged by source type, rather than silencing one in favor of the other. For any calculation step being checked, the retrieval layer surfaces the specific regulatory provision governing it, and, where one exists, the internal interpretation memo addressing that same provision — with both explicitly labeled as what they are, not merged into a single undifferentiated "guidance" passage. The comparison logic then runs against both: does the calculation match the regulatory text as written, and separately, does it match the internal interpretation. When those two answers agree, the check is clean. When they diverge — the calculation follows the internal interpretation but that interpretation is itself a stretch from the literal regulatory text — that divergence is exactly the kind of finding a validation system exists to surface, not suppress.

This produces a three-way verdict richer than a simple match/mismatch: a calculation can match both sources, match the internal interpretation but diverge from a strict reading of the regulation, or diverge from both. The middle case is the one unique to this domain and the one a less careful retrieval design would collapse into a false negative (if it only checks against internal interpretation) or a false positive flood (if it only checks against regulatory text). Making that middle case visible, with both source passages cited, gives the human reviewer — who is positioned to know the institutional history behind a given interpretation — exactly what's needed to judge whether the divergence is a known, accepted position or something that needs re-examination.

## Provenance Metadata as a Retrieval-Time Concern

Because this three-way comparison depends entirely on knowing which retrieved passage is regulatory text and which is internal interpretation, provenance tagging has to be enforced at ingestion and carried through retrieval as a structural property of each chunk — not something inferred from content or left to the model to figure out from context. Each document entering the corpus is tagged with its source type (regulatory text vs. internal interpretation), its effective-date range, and, for internal interpretations, the governance body or committee that approved it, since "who approved this interpretation and when" is itself relevant information for a reviewer weighing whether a divergence is a settled, sanctioned position or something that's drifted without proper sign-off.

This is a stricter version of the source-attribution principle from the stress-testing RAG lecture: there, the risk was conflating two versions of the *same kind* of document across cycles. Here, the risk is conflating two *different kinds* of document that happen to address the same topic and can genuinely disagree. The retrieval system's job isn't just finding relevant passages — it's preserving the distinction between what a rule says and what the bank has decided the rule means, all the way through to the point where a human makes the final call.

## Architecture Notes

This use case pushes the **Context** layer's provenance principle further than any other in the course: it's not enough to track where information came from, the system has to actively preserve and surface a genuine disagreement between two legitimately-sourced inputs rather than letting either one silently win. That three-way verdict structure is also a **Prompt** layer design choice — the comparison prompt has to be built to produce "matches regulation but diverges from internal interpretation" (and its inverse) as first-class output categories, not just squeeze every result into a binary match/mismatch that would erase the exact distinction this domain depends on.
