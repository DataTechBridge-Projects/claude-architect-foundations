---
title: "Lab: Tool Ecosystem Design Simulator"
parent: The Tool Layer
nav_order: 12
duration: "1:36"
---

# Lab: Tool Ecosystem Design Simulator
{: .no_toc }

*Video duration: 1:36*

This lab hands you a scenario rather than a checklist: an internal support team wants a Claude-based agent that can look up orders, check shipping status, issue refunds, and update customer records across three underlying systems, two of which already have community-maintained MCP servers available and one that doesn't. Your job is to design the actual tool set — not the prompt, not the agent's autonomy level, just the tools themselves.

For each capability, you choose the granularity (one broad tool or several narrow ones), write the name and description as if a model with no other context will read only that text, and specify the schema, including which fields are structured and which, if any, are legitimately free-text prose. You also decide, for each tool, what its error responses should say for its two or three most likely failure modes, and whether those failures are retryable.

The simulator then runs your tool set against a batch of realistic requests and shows you where the model — reading only what you wrote — picked the wrong tool, filled in a bad argument, or got stuck after a failure with nowhere to go. There's no single passing design; the exercise is built so an overly fragmented tool set and an overly consolidated one both produce visible selection errors, and the score rewards the same judgment this module has been building toward: matching structure, naming, and error design to how a reasoning model actually reads them.

The scenario is deliberately built to force every principle from this module into the same design pass. The two systems with existing MCP servers push you to reason about the MCP governance material directly — do you trust the community-maintained server as-is, or does its `update_customer_record` tool need a scoped-down credential before it's safe to connect, given that a support agent should never need the broader write access the vendor's server happens to expose? The third system, with no server available, forces you to write tool definitions and infrastructure from scratch, which is exactly where the naming, description, and schema discipline from earlier in this module gets tested without a vendor's defaults to lean on. And the refund capability specifically is where the structure-versus-prose call matters: an `amount` and `reason` that should be typed and enumerated, alongside a `customer_note` that legitimately needs to stay free text, mirrors the split covered earlier in the module almost exactly.

A typical run surfaces failures in recognizable clusters. Teams that split "look up order" into four near-identical tools (`get_order_by_id`, `get_order_by_email`, `get_order_by_phone`, `get_order_by_date`) usually see the simulator's model calling the wrong variant when a request supplies two identifiers at once, or hesitating across all four when only a partial description is available — a direct illustration of the granularity tradeoff. Teams that consolidate too aggressively into one `manage_order` tool with a dozen optional parameters usually see technically-valid but semantically-wrong calls, where the model fills in a parameter combination that passes schema validation but doesn't do what the request actually asked for. Neither failure is a "the model is dumb" problem — both trace back to a definition choice you made, which is the point: the lab is a low-stakes place to make those mistakes and see their consequences immediately, rather than discovering them the first time a real deployment produces a bad refund or a missed shipping update.

<!-- prevnext:start -->

---

| [&larr; Previous: Different Workflows, Similar Principles](11-different-workflows-similar-principles) | [Next: Module Outro &rarr;](13-module-outro) |
|:---|---:|

<!-- prevnext:end -->
