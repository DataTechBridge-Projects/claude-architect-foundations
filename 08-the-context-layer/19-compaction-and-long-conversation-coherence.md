---
title: Compaction and Long-Conversation Coherence
parent: The Context Layer
nav_order: 19
duration: "13:00"
---

# Compaction and Long-Conversation Coherence
{: .no_toc }

*Video duration: 13:00*

"Managing the Context" introduced compaction alongside summarization and discarding as one of the three tools an architect reaches for once a session's history grows large enough to threaten the context budget. This lecture stays with compaction specifically and goes deeper into the risk that makes it genuinely difficult rather than mechanical: every act of compaction is a bet that what got trimmed away didn't matter, and a long enough conversation will eventually test that bet against something that did.

## What compaction is, precisely

**Compaction** is the practice of reducing the size of accumulated conversation history — by summarizing, trimming, or otherwise compressing older material — so that a long-running session can continue operating within its context budget rather than running out of room or drowning in low-value accumulated detail. Left alone, a session's history only grows; every turn adds more transcript, and nothing about the mechanics of a conversation prunes it automatically. Compaction is the deliberate intervention that keeps history from becoming the dominant, ever-expanding consumer of the window described in "How the Context Window Works."

The reason compaction deserves its own lecture, beyond the introduction it got earlier in this module, is that its central risk — losing coherence — is not a rare edge case. It is the *default* outcome of compressing anything: information is lost whenever you make something smaller, and the entire question that matters is *which* information, and whether that information turns out to be load-bearing later. A session that compacts aggressively and never encounters a moment where the trimmed detail was needed looks, from the outside, identical to one that never had a coherence problem at all — right up until the one turn where a discarded specific fact, a subtle qualifying detail, or an earlier commitment the user made turns out to matter, and the system either contradicts itself, repeats a mistake it already worked through, or simply seems to have forgotten something it was explicitly told.

{: .important }
> **Losing information is the default outcome of compaction, not a rare edge case** — the only question is which information, and whether it turns out to be load-bearing later. A session that compacts aggressively and never hits a problem looks identical, from the outside, to one that never had a coherence risk at all, until the turn that needed the trimmed detail.

## The coherence risks, named specifically

Three concrete failure patterns recur across real compaction failures, and it's worth being able to name each one rather than lumping them together as "losing detail."

**Losing specific facts.** A compacted summary that says "the customer's issue was resolved" is technically accurate but has discarded the actual figure, date, or identifier that a later turn might need to reference precisely — an order number, an exact refund amount, a specific error code. General summaries are good at preserving the shape of what happened and bad, by construction, at preserving the specific values embedded in it, unless the compaction step is deliberately told to protect them.

**Losing subtle details.** Some information matters not because of its specificity but because of a qualifier attached to it that's easy to summarize away. "The fix worked, but only for the Windows build — it still fails on Linux" compacts dangerously easily into "the fix worked," silently dropping the one clause that actually mattered for anyone deciding whether to close the issue. Subtlety-loss is more dangerous than fact-loss in one specific way: a missing fact is often noticeably missing, prompting someone to go looking for it, while a missing qualifier leaves behind a statement that still reads as complete and confident, giving no signal that anything was dropped.

**Losing earlier commitments.** A long conversation often contains commitments — the user asked for a particular approach, the agent agreed to a particular constraint, a decision was made three turns in that should still bind at turn thirty. A compaction pass focused on summarizing outcomes can drop the fact that a commitment was ever made at all, especially if it was stated conversationally rather than flagged as a decision. The failure surfaces later as an agent contradicting an instruction it was actually given, not because it disregarded the instruction, but because the instruction never survived into the compacted history in the first place.

All three failure patterns share a root cause: compaction is usually optimized to preserve *what happened*, and all three kinds of loss involve something that isn't well captured by "what happened" alone — a precise value, a limiting condition, or a standing constraint.

## Strategies: filter, summarize, reference

Rather than treating compaction as one blunt operation, it helps to separate it into three distinct strategies, each suited to different material, and to recognize that a good compaction pass usually applies different strategies to different parts of the same history rather than picking one for everything.

**Filter** means dropping content that's simply irrelevant to anything that follows — a clarifying exchange that resolved cleanly and has no bearing on later decisions, an exploratory tool call that turned out to be a dead end with no useful residue. Filtering is appropriate exactly when the content's absence changes nothing, which is a judgment call worth making deliberately rather than assuming by default; content that looks irrelevant in the moment sometimes turns out to matter later precisely because its relevance wasn't obvious until then.

**Summarize** means compressing content into a shorter restatement that preserves its substance while discarding its verbosity — the technique covered in depth in "Managing the Context," including the discipline of preserving decisions and their justification rather than just outcomes. Summarization is the right tool for genuinely voluminous material whose *conclusion* matters more than its blow-by-blow path to that conclusion: a long debugging exchange, an extended negotiation over an approach, a lengthy tool result that's already been acted on.

**Reference** is the strategy most distinct from the other two, **and arguably the most underused**: instead of compressing content into a shorter restatement, replace it with a pointer to where the full content can still be retrieved if it's needed — a file path, a document ID, a note that "the full account history is available via the lookup tool if needed again." Referencing trades a small amount of upfront convenience (the full detail isn't sitting in context, ready to hand) for something summarization can't offer: the ability to recover the *exact* original content on demand, rather than only ever having access to whatever a past summarization pass judged worth keeping. Referencing is the natural choice for content that's large, rarely needed again, but occasionally needed in full — exactly the profile where summarizing risks the fact-loss and subtlety-loss failure modes above, but discarding entirely is too risky given how consequential the content might be if it does turn out to matter.

| Strategy | What it does | Best for |
|---|---|---|
| Filter | Drops content whose absence changes nothing | Clearly irrelevant material, cleanly resolved exchanges |
| Summarize | Compresses content into a shorter restatement | Voluminous material where the conclusion matters more than the path to it |
| Reference | Replaces content with a pointer to the full original | Large, rarely-needed material that's occasionally load-bearing |

## Deciding what's safe to compact vs. what must survive verbatim

The strategic question underneath all three techniques is the same one "Managing the Context" raised about summarization generally: how reversible is being wrong here? Content backing a decision that's easy to revisit or correct later tolerates aggressive compaction — filtering, or a lossy summary — because the cost of having compacted away a detail is just re-deriving it, at worst. Content backing a decision that's expensive, embarrassing, or impossible to reverse — a number a downstream irreversible action depends on, an explicit commitment the user is relying on the system to honor, a policy citation justifying a consequential judgment call — deserves a materially higher bar before it's compacted at all, and when it is compacted, it deserves referencing or careful, fact-preserving summarization rather than a general-purpose pass that optimizes for narrative brevity.

This is precisely where compaction intersects with provenance, covered in "Data Provenance and Conflicts": the same lecture observes that summarization is one of the most common places provenance metadata quietly disappears, and the fix there generalizes directly here — before compacting a load-bearing fact away, ask not just "do I still need this" but "will whoever reads this later still be able to tell how much to trust it, and to recover the original if they need to." A scratchpad, covered in the previous lecture, is one concrete way to protect exactly this kind of load-bearing state from an overly aggressive compaction pass, by giving it a durable home the compaction process is designed to respect rather than casually summarize away.

**Why it matters for the exam:** Expect scenario questions describing a long-running session where a compaction pass caused a downstream error — an agent contradicting an earlier commitment, repeating a ruled-out approach, or losing a precise figure — and ask you to identify the failure as a coherence risk of compaction, then choose the right remedy among filter, summarize, and reference. The exam rewards recognizing that reference (a pointer to full content) is often the safer choice for large, occasionally-needed material, over defaulting to summarization for everything.

<!-- prevnext:start -->

---

| [&larr; Previous: The Scratchpad Pattern: A Durable Memory Slot](18-the-scratchpad-pattern-a-durable-memory-slot) | [Next: Calibrating Confidence: Validation Sets and Stratified Sampling &rarr;](20-calibrating-confidence-validation-sets-and-stratified-sampli) |
|:---|---:|

<!-- prevnext:end -->
