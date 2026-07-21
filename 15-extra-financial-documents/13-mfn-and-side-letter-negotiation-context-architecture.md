---
title: MFN & Side Letter Negotiation - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Financial Documents"
nav_order: 13
duration: "6:45"
---

# MFN & Side Letter Negotiation - Context Architecture
{: .no_toc }

*Video duration: 6:45*

Give an agent genuine autonomy to widen its own search as it discovers relevant material, as the previous lecture describes, and the context architecture question becomes correspondingly harder: what should be loaded up front, what should be pulled in on demand, and how do you keep an open-ended comparison task from either missing something material or drowning in irrelevant side letters it didn't need to read.

## The full population of side letters, or a retrieved subset

The honest answer is that it depends on fund size, and the architecture should support both modes rather than committing to one. For a fund with a modest number of side letters — a few dozen — it's often feasible to load structured summaries of the *entire population* into context up front: not the full text of every side letter, but a compact, per-letter summary capturing investor identity, whether an MFN clause exists, and a short structured digest of what that MFN clause covers and excludes. That gives the agent a map of the whole terrain before it starts reasoning, which is valuable precisely because MFN analysis benefits from seeing the full pattern of what's been granted across the fund, not just the letters that superficially look relevant to the term at hand.

For a larger fund with a hundred-plus side letters, loading full text for all of them isn't practical, and the architecture shifts toward the retrieval pattern developed in the next lecture: a summarized index of the whole population (which is cheap enough to keep in context even at scale, since a structured one-paragraph digest per letter is a small fraction of the letter's full text) paired with on-demand retrieval of full clause text for any letter the agent's initial pass over the summaries flags as a plausible candidate. The summarized index functions like a map, and the agent decides, based on what the map shows, which specific territories are worth pulling in full detail — which is exactly the agentic behavior the previous lecture argued for for a fixed workflow.

Either way, the architectural principle is the same: the agent should always have visibility into the *shape* of the full population — how many side letters exist, which have MFN clauses, broadly what categories of terms each covers — even when it doesn't have the full text of every one loaded at once. Losing sight of the population's shape is what causes an agent to under-search: if it only ever sees the letters a first-pass keyword match happens to surface, it has no way of knowing there might be other relevant ones it hasn't looked at.

## The proposed new terms

The second, smaller but equally essential context component is the newly proposed term itself, framed precisely: what's being offered, to which investor, and in what form (fee terms, reporting rights, co-investment access, excuse rights, and so on). This needs to be represented with enough structure that it can be meaningfully compared against the structured digests of existing MFN clauses — a vague framing like "give Investor X a better deal" doesn't give the agent anything precise to compare, while "reduce Investor X's management fee from 2.0% to 1.75% on committed capital above $50M" gives it a comparison target with the specificity MFN analysis actually requires.

## MFN trigger definitions

The third component, and the one most specific to this use case, is a working reference for how MFN clauses are typically structured and what kinds of qualifiers commonly limit them — comparability-by-investor-size language, time-window limitations (MFN rights are often only exercisable within some window after a term is granted to someone else), category carve-outs (an MFN clause might explicitly exclude co-investment rights or excuse provisions from its scope), and notice-versus-automatic-election mechanics (some MFN clauses require the GP to affirmatively notify eligible investors, others let an investor request comparable terms and require the GP to respond). This isn't a fixed legal answer key — every side letter's actual language governs its own terms — but a general reference framework that helps the agent recognize *which category* of qualifier it's looking at when it reads a specific clause, the same way a glossary of legal doctrine helps a junior associate orient quickly rather than parsing every clause from complete first principles.

## Keeping population-wide context from becoming an unfocused blob

The risk specific to this use case, given that context now includes a whole population of comparison documents rather than one reference document, is that the working context degrades into an undifferentiated pile of side-letter language that makes it harder, not easier, for the agent to reason clearly. The mitigation is the same discipline this module has used throughout: everything in context stays labeled by source and role — this is Investor A's side letter, this is Investor A's MFN clause specifically, this is the newly proposed term under review, this is the general MFN-qualifier reference — so the agent's comparisons are always between clearly identified, correctly attributed pieces of text, never between an unlabeled blend of "side letter language" that's lost track of which investor's terms are which. That labeling discipline matters more here than in the templated use cases precisely because the agent is actively reasoning across many documents at once, rather than processing one document in isolation.

**Architecture Notes:** This lecture applies the Context layer's "what does the model actually see" question to a genuinely open-ended comparison task: a summarized population-wide index paired with on-demand full-text retrieval is the concrete technique for giving an agent visibility into scope without paying the cost of loading everything. Strict source labeling across many side letters at once is the same data-provenance discipline as the LPA module's investor-isolation requirement, adapted to a task where cross-document comparison is the point rather than the risk to avoid.
