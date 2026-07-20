---
title: Multiple Pathways
parent: The Tool Layer
nav_order: 5
duration: "0:32"
---

# Multiple Pathways
{: .no_toc }

*Video duration: 0:32*

A tool set designed around one assumed path to a goal breaks the moment reality doesn't match that assumption — a lookup-by-ID tool is useless when the caller only has a name. Good tool design often deliberately offers more than one way to reach the same outcome: search alongside direct lookup, a broad tool alongside a narrow one, a fallback that works with partial information. This isn't redundancy for its own sake — it's resilience against the gap between what you assumed the model would know at call time and what it actually has available in context.

Consider a customer-support agent working an order. If the only available tool is `get_order_by_id`, the agent is stuck the moment a customer says "my package never arrived" without quoting an order number — a completely ordinary way for a real conversation to unfold. Pairing that tool with `search_orders_by_customer` (accepting an email, name, or approximate date) turns a dead end into a two-step recovery: search first, narrow to the right order, then proceed. Neither tool is redundant; each covers a distinct starting condition the model might find itself in, and the model's job becomes picking the pathway that matches what it actually has, rather than failing outright because the one tool it was given assumed a fact not in evidence.

It's worth being explicit that this principle pulls in tension with the granularity guidance covered elsewhere in this module: adding pathways adds tools, and every additional tool is another boundary the model has to reason about correctly relative to its neighbors. The resolution isn't "always add more paths" — it's recognizing when a missing pathway represents a real, recurring gap between assumed and actual caller state, versus when it would just be a rarely-needed variant cluttering the tool set. A second lookup pathway earns its place when the condition it handles is common enough that the tool set's current design would otherwise fail routinely, not merely occasionally. This same tension resurfaces in this module's granularity discussion, and it's also why an MCP server pulled from a vendor's own API often needs a lightweight custom tool alongside it — the vendor designed for their own typical caller, not for the partial-information reality your agent actually operates in.
