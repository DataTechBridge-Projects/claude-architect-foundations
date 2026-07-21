---
title: Key Distinctions - The Context Layer
parent: The Context Layer
nav_order: 23
duration: "4:24"
---

# Key Distinctions - The Context Layer
{: .no_toc }

*Read duration: 4:24*

This module covers a lot of ground, but most of its exam-relevant judgment calls reduce to a short list of either/or distinctions. Knowing which side of each one a scenario is actually testing is usually the fastest path to the right answer.

**Summarization vs. compaction vs. discarding.** Summarization rewrites verbose content into a shorter narrative that preserves decisions and their justification. Compaction removes or compresses redundant and superseded material while keeping structure intact — closer to garbage collection than editorial rewriting. Discarding drops content outright with no path back to it. When a question describes losing detail from context, identify which of the three actually happened before judging whether it was the right call.

**Threshold-based vs. milestone-based compaction.** Threshold-based compaction triggers on a token budget crossing some proportion of the limit; milestone-based compaction triggers on the completion of a meaningful unit of work. Milestone-based is generally preferable when task structure has clear seams, because it avoids compressing something mid-thought.

**Context isolation vs. context sharing between agents.** A subagent scoped to a purpose-built context performs more predictably and avoids noise contamination, but over-isolation can starve a subagent of information it genuinely needs. The right boundary is deliberately drawn around what a specific subagent's task requires — never "isolate everything" or "share everything" as a blanket rule.

**Guess-and-continue vs. flagged assumption vs. clarifying question vs. escalation vs. full stop.** This is the core spectrum from the escalation lecture. Guess-and-continue (silent resolution) is almost never correct. The other four are legitimate, and choosing among them is a granularity judgment matched to the stakes and reversibility of the specific decision — not a single "safe" default that applies everywhere.

**A single wrong answer vs. a propagated error.** A single wrong answer is usually visible and easy to catch on its own. A propagated error is camouflaged by everything reasonable that got built on top of it, which is what makes it more dangerous and harder to detect from the final output alone.

**Checkpoints vs. re-verification vs. provenance tracking.** Checkpoints create deliberate pauses to re-examine load-bearing conclusions. Re-verification actively re-confirms a specific conclusion against ground truth before acting on it. Provenance tracking attaches source, currency, and trust metadata so both of the other techniques know what to prioritize. They're complementary, not interchangeable — a scenario testing "what should have caught this" usually has one specific right answer among the three.

**Retrieval-driven context vs. CLAUDE.md-style static context.** Static, always-loaded context suits small, stable, broadly-relevant material. Retrieval-driven, search-based narrowing suits large or heterogeneous corpora where no small static subset is broadly relevant. Applying the wrong one — an elaborate retrieval pipeline for a small project, or a bloated always-loaded file for a huge one — is itself a common wrong answer.

**File-level vs. symbol-level context selection.** Whole files preserve surrounding context at higher token cost; symbol-level extraction is cheaper but risks stripping context that clarified how a symbol should be used. The right choice tracks how cohesive the file actually is, not a fixed preference.

**A review checkpoint that exists vs. a review checkpoint that functions.** A gate with poorly scoped context — too little or overwhelmingly too much — exists on paper without actually catching errors. A well-designed gate shows the decision (not the whole session), surfaces uncertainty explicitly, and offers granular response options, not just approve/reject.

**Recency vs. authority when sources conflict.** "More recent" is a good tiebreaker for volatile facts and a bad one for stable facts restated imprecisely. Neither recency nor confident phrasing is a universal rule — real resolution weighs source, currency, and verifiability together, and an unresolvable conflict should be surfaced, not silently picked.

**Why it matters for the exam:** When a scenario question feels ambiguous about what it's testing, check it against this list first — most Context Layer questions are one of these distinctions wearing a business scenario as a costume.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Outro](22-module-outro) | [Next: Quiz Prep: The Context Layer (20 Questions) &rarr;](24-quiz-prep-the-context-layer-20-questions) |
|:---|---:|

<!-- prevnext:end -->
