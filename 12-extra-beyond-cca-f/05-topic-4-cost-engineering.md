---
title: "Topic 4: Cost Engineering"
parent: "Extra Module: Beyond the CCA-F"
nav_order: 5
duration: "0:24"
---

# Topic 4: Cost Engineering
{: .no_toc }

*Video duration: 0:24*

Agentic loops multiply cost fast, in a way that surprises teams coming from a single-call mental model. Every tool-result-then-model-call round trip resends the growing conversation history, since each new model call needs the full context to that point — so a ten-step loop doesn't cost roughly ten times a single call, it costs the sum of ten increasingly large context windows, which can be dramatically more. Understanding that shape is the first step in cost engineering, because it means the biggest lever is very often architectural, not a pricing negotiation.

A few techniques recur across production systems. **Model tiering** routes work to the cheapest model capable of doing it well: a small, fast model for classification, routing, or simple extraction, reserving the most capable (and most expensive) model for the step that genuinely needs its reasoning. **Prompt caching** avoids re-paying for stable, repeated context — a CLAUDE.md file, a large tool-definition block, a system prompt — that's identical across many calls in a session or across many sessions. **Context compaction**, covered in depth in the Context Layer module, does double duty here: summarizing or trimming history to keep the model's working set relevant also directly shrinks the token count you're paying for on every subsequent call. And for work that doesn't need an immediate response, **batch APIs** trade latency for a substantial cost discount, which is often a good trade for background or bulk-processing workloads.

| Technique | What It Does | Helps Most When |
|---|---|---|
| Model tiering | Routes work to the cheapest capable model | Steps vary widely in reasoning difficulty |
| Prompt caching | Avoids re-paying for stable, repeated context | A CLAUDE.md, tool block, or system prompt repeats across calls |
| Context compaction | Trims/summarizes history to shrink token count | Long-running loops with growing conversation history |
| Batch APIs | Trades latency for a cost discount | Background or bulk-processing work with no immediate-response need |

The architectural lesson underneath all of this: cost isn't a deployment afterthought you tune after the system works. It's a direct, mechanical consequence of the context management and agentic-loop decisions covered earlier in this course — a system designed with unbounded context growth and no tiering will be expensive by construction, no matter how good the pricing is per token.

<!-- prevnext:start -->

---

| [&larr; Previous: Topic 3: Agent Performance Evaluation](04-topic-3-agent-performance-evaluation) | [Next: Topic 5: Security Architecture &rarr;](06-topic-5-security-architecture) |
|:---|---:|

<!-- prevnext:end -->
