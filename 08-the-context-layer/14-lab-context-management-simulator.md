---
title: "Lab: Context Management Simulator"
parent: The Context Layer
nav_order: 14
duration: "1:32"
---

# Lab: Context Management Simulator
{: .no_toc }

*Video duration: 1:32*

This lab puts you on the other side of the problem this module has spent its lectures describing: you'll manage a simulated context window that starts small and grows, turn by turn, under realistic pressure. Each round introduces new material competing for space — a tool result, a retrieved document, a user message, a subagent's report — and you have to decide, with a visible token budget ticking down, what to keep verbatim, what to summarize, what to compact, and what to discard outright, exactly as described in the "Managing the Context" lecture.

The twist is that some of the incoming material is deliberately unreliable: a few pieces carry weak or conflicting provenance, a few are stale by the time they arrive, and a couple are genuinely ambiguous instructions with no clean resolution in the data you're given. Your simulated agent's downstream decisions depend entirely on what you chose to keep, how you labeled its trustworthiness, and whether you flagged an assumption or escalated instead of guessing. The simulator scores you not just on whether you stayed under budget, but on whether a mistake you let slide in round three quietly wrecked round nine — the error-propagation lesson, experienced firsthand rather than just read about.

Structurally, the lab runs as a sequence of rounds, each presenting one or two new inputs alongside a shrinking visible token budget and a small set of actions: keep verbatim, summarize, compact, discard, or tag-and-hold for later verification. Every action you take is logged, and the simulator carries your choices forward exactly the way a real session would — **a fact you compacted away in round two is genuinely gone by round nine**, not quietly still available if it turns out you needed it. That's deliberate. The most common lesson people report from this lab isn't about summarization mechanics at all; it's the visceral experience of realizing, several rounds later, that a discard decision that felt safe in the moment removed the one piece of context that would have caught a downstream contradiction.

Several rounds are built specifically to exercise other lectures in this module rather than "Managing the Context" alone. One round hands you two contradicting facts with different-looking sources and forces you to decide whether to weigh them, escalate, or — the wrong answer the simulator is specifically watching for — silently pick whichever one sounds more recent or more confident, which is exactly the naive resolution the provenance-and-conflicts lecture warns against. Another round gives your simulated agent a decision point with only three listed options, none of which honestly fit the situation, to see whether you notice the missing do-nothing-and-ask option before the agent forces a bad pick. A later round varies the simulated task's stakes explicitly — sometimes you're managing context for a low-stakes drafting task, sometimes for something framed as irreversible — to test whether your keep/discard choices actually shift with failure affordance the way they should, or whether you apply the same level of caution regardless of what's at stake.

At the end, the simulator produces a short report rather than just a pass/fail score: which decisions were fine, which technically stayed under budget but destroyed load-bearing provenance, and which mistake — if any — propagated furthest before causing visible downstream damage. Treat that report as the real deliverable of the lab. The token-budget math is the easy part; the judgment calls about what's safe to compress, what's safe to drop, and what has to be escalated rather than guessed at are the actual skill this entire module has been building toward, and this lab is where you find out, under some real time pressure, whether it's landed.

<!-- prevnext:start -->

---

| [&larr; Previous: Putting the "Provenance" in Data Provenance](13-putting-the-provenance-in-data-provenance) | [Next: How the Context Window Works: Tokens, Limits, and Tradeoffs &rarr;](15-how-the-context-window-works-tokens-limits-and-tradeoffs) |
|:---|---:|

<!-- prevnext:end -->
