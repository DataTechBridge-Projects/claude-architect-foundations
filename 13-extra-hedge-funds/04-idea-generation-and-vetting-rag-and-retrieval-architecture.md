---
title: Idea Generation/Vetting - RAG.& Retrieval Architecture
parent: Extra Module: Solution Architecture Patterns - Hedge Funds
nav_order: 4
duration: "9:01"
---

# Idea Generation/Vetting - RAG.& Retrieval Architecture
{: .no_toc }

*Video duration: 9:01*

Context architecture decides what's *allowed* into the window at each stage; retrieval architecture decides *how the system finds* the right material to fill that allowance. For an idea generation and vetting system, retrieval is not a single index behind a single search box — it's several distinct corpora with different update cadences, different retrieval strategies, and, most importantly, different *retrieval objectives* depending on which agent is asking. Getting this layer wrong is the single most common way these systems quietly degrade into producing fluent, well-cited-looking output that's actually built on thin or stale evidence.

## What Gets Indexed

Four broad corpora feed this system, and each deserves its own indexing treatment rather than being flattened into one generic document store.

**Research notes** — the desk's own prior work, including both published theses and the analyst commentary that never became a formal thesis — are the highest-value, lowest-volume corpus. These are indexed with rich metadata: the name or sector covered, the date, the outcome if known, and tags for which risk factors were implicated. Because this corpus is small relative to public filings and news, it can support fairly literal, high-precision retrieval rather than needing aggressive approximate search.

**Regulatory filings** (10-Ks, 10-Qs, 8-Ks, proxy statements, and equivalent filings for non-US names) are large, dense, and highly structured documents where the useful signal is often a specific paragraph, not the whole filing. These are chunked at a granularity that respects the filing's own structure — by section and sub-item rather than by a fixed token count — so that a retrieved chunk about "Item 1A Risk Factors" doesn't get sliced across an arbitrary boundary that separates a claim from its qualifying language. Filings are indexed with strict version and amendment metadata, since a company can and does refile.

**News and market commentary** is the highest-volume, fastest-moving, and lowest-average-signal corpus. It benefits from a two-stage retrieval approach: a fast, recency-weighted first pass that narrows a huge stream down to what's plausibly relevant to the candidate at hand, followed by a more careful semantic pass over that narrowed set. Indexing news in full is often not worth the cost — a rolling window (weeks for daily commentary, longer for more consequential single events) keeps the index from growing without bound while still covering the horizon a fast-moving trade thesis actually cares about.

**Transcripts** (earnings calls, investor days, conference appearances) sit between filings and news in both structure and volatility. They're chunked by speaker turn and topic segment rather than by raw length, which matters a great deal here because the *way* an executive answers a question — hedging language, a pivot away from a direct answer — is frequently more informative than the literal content, and chunking that splits a question from its answer destroys exactly that signal.

## Retrieval Strategy: Different Objectives for Different Agents

The most important retrieval design decision in this system is that the generation agent and the vetting agent are not just fed different context, as covered in the previous lecture — they issue fundamentally different *kinds* of retrieval queries against the same underlying corpora. The generation agent's retrieval objective is roughly "find evidence that helps build and support this thesis": it searches for corroborating fundamentals, comparable past setups that worked, and language that reinforces the pattern it's investigating. The vetting agent's retrieval objective is close to the opposite: "find evidence that would make this thesis wrong." That means explicitly querying for the strongest counter-arguments, not just running the same search and taking a second look at the same results — a semantic search for "reasons to buy X" and a semantic search for "reasons X is overvalued" or "risks to X's thesis" surface materially different documents even against an identical corpus, and a vetting agent that only reruns the generation agent's queries will simply confirm what it already found.

This asymmetric-query approach is the retrieval-layer expression of the same principle that motivated splitting generation and vetting into separate agents in the first place: a system that only ever asks confirming questions produces confirming answers, regardless of how good its underlying retrieval index is. Structurally, this is implemented by giving the vetting agent an explicit instruction and, ideally, a dedicated tool or retrieval mode tuned to surface disconfirming or contradicting evidence — for instance, retrieving documents that discuss the same company or sector but were excluded from the generation agent's top results, or explicitly searching for short-seller commentary and skeptical analyst notes where the generation agent's search terms would naturally have skewed toward supportive coverage.

## Freshness and Recency Handling

Market-relevant retrieval has a freshness problem that most enterprise RAG systems don't: the *rank order* of relevance genuinely changes hour to hour, not just document contents changing. A retrieval system that treats all indexed documents as equally current, weighted only by semantic similarity to the query, will happily surface a six-month-old article that's semantically close to the query text but has been overtaken by events. The fix is a retrieval scoring function that blends semantic relevance with an explicit recency decay, tuned differently per corpus: aggressive decay for news and market commentary, where anything older than a few days is usually noise for an active thesis; moderate decay for transcripts and filings, where the most recent quarter's filing dominates but an older filing might still be relevant for establishing a trend; and little to no decay for research notes and past thesis history, where an idea's age is informative on its own terms rather than something to be down-weighted away.

Equally important is treating retrieval *metadata* as part of the retrieved content, not just a sorting key. Every chunk that reaches an agent's context carries its source, its publication or filing date, and — for filings specifically — whether it has been superseded by a later amendment. Agents are prompted to surface this metadata in their output rather than silently absorbing it, so a thesis's citations read as "per the 10-Q filed March 2026" rather than an unattributed claim, giving a human reviewer the ability to judge staleness themselves rather than trusting the system's internal freshness scoring blindly.

Finally, retrieval for this system deliberately keeps a **negative results** discipline: if the vetting agent's targeted search for contradicting evidence comes back thin, that absence is itself reported rather than silently treated as "no contradicting evidence exists." A sparse-evidence result and a genuinely clean thesis look identical unless the system distinguishes them explicitly, and conflating the two is exactly the kind of failure that lets a fragile thesis look more vetted than it actually is.

## Architecture Notes

Retrieval here is doing more than fetching relevant documents — the asymmetric query design between generation and vetting is a Prompt-layer and Context-layer decision expressed through retrieval mechanics, ensuring the two agents genuinely see different evidence rather than the same evidence twice. Recency-weighted scoring and mandatory source/date metadata are this use case's answer to the Context layer's provenance discipline: in a fast-moving market, a retrieved fact without a timestamp is not meaningfully different from a fact with no source at all.
