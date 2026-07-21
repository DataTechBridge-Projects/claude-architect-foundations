---
title: Results Polluting the Context?
parent: The Agentic Layer
nav_order: 5
duration: "0:21"
---

# Results Polluting the Context?
{: .no_toc }

*Video duration: 0:21*

Step 5 of the orchestration loop — appending the tool result to context — has a trap: tool results aren't always small. A search returning thousands of matches, a file read dumping an entire log, or a raw API response with dozens of unused fields all get appended verbatim by a naive implementation, and every one of those tokens is re-sent, and re-billed, on every subsequent model call. Left unchecked, results accumulate faster than the actual task justifies, crowding out useful context and driving up cost. Filtering, truncating, or summarizing results before they re-enter context is a core Tool- and Context-layer responsibility, not an afterthought.

Consider a concrete case: an agent asked to find where a bug originates runs a grep-style search across a codebase and gets back four thousand matching lines. A naive tool wrapper appends all four thousand lines to the transcript verbatim. The model only actually needed the dozen or so matches near the function it's investigating, but now every one of the next ten loop iterations re-sends those four thousand lines as part of the growing history, even after the model has clearly moved on to reasoning about something else. The result: ballooning per-call cost, slower generation (more tokens to process before the model even starts producing its next response), and — often worse than the cost — a diluted signal-to-noise ratio that makes it harder for the model to locate the few tokens that actually matter amid thousands that don't.

The fix isn't a single technique but a family of them, chosen based on what the result actually contains. **Truncation** caps a result at some reasonable size and tells the model more is available on request — useful for logs and long file reads. **Filtering** returns only entries matching some relevance criterion instead of everything a query technically found — useful for search results. **Summarization** compresses a large result into a distilled description, at the cost of losing recoverable detail — useful when the specifics genuinely don't need to persist, only the conclusion drawn from them. A well-designed tool doesn't wait for the model to ask for less; it's built from the start to return only what's proportionate to the request.

| Technique | What it does | Best suited for |
|---|---|---|
| Truncation | Caps result size, signals more is available | Logs, long file reads |
| Filtering | Returns only relevance-matching entries | Search results |
| Summarization | Compresses into a distilled description | Cases where only the conclusion matters |

This connects directly to the multi-agent coordination pattern covered earlier in this module: dispatching a subagent to do the noisy exploration and report back only a distilled summary is, in effect, the same fix applied at a coarser grain — instead of filtering a single tool result, you're filtering an entire subagent's worth of intermediate work down to the report that actually re-enters the orchestrator's context. Both are the same underlying discipline: keep only what's proportionate to the task, not everything a system was technically capable of producing.

<!-- prevnext:start -->

---

| [&larr; Previous: Structured Tool Use](04-structured-tool-use) | [Next: Multi-Agent Coordination &rarr;](06-multi-agent-coordination) |
|:---|---:|

<!-- prevnext:end -->
