---
title: Let's Coin Context Data Governance (CDG)
parent: The Context Layer
nav_order: 3
duration: "0:32"
---

# Let's Coin Context Data Governance (CDG)
{: .no_toc }

*Video duration: 0:32*

It's worth putting a name on a discipline this module keeps circling: deciding what data is allowed into an agent's context, from what source, and under what conditions. Call it **Context Data Governance (CDG)** — not an official Anthropic term, just a useful label for the recurring set of questions underneath everything from `CLAUDE.md` scoping to retrieval design to provenance tracking. Whenever you're asking "should this be in the window, and do I trust where it came from," you're doing CDG, whether or not you ever use the acronym again.

Naming it matters because, without a name, this work tends to happen implicitly and inconsistently — one engineer scopes a subagent's context carefully, another bolts on a retrieval pipeline without ever asking what happens when it returns something wrong, and nobody notices they were solving the same underlying problem with wildly different levels of rigor. A named discipline gets a checklist, a review step, an owner. Treat CDG as covering at least four concrete questions for any given piece of context: **admission** (does this source earn a place in the window at all, and under what conditions), **trust classification** (verified, stated-but-unconfirmed, or inferred — the same categories the provenance lecture later in this module formalizes), **retention** (how long does this stay before it's summarized, compacted, or discarded), and **boundary** (does this cross from one agent's context into another's, and if so, filtered or raw).

A concrete example makes this less abstract. A support-escalation agent's CDG policy might state: account-lookup tool results are admitted as verified and timestamped; the customer's own claims about their account are admitted but tagged stated-unverified; `CLAUDE.md` policy content is reviewed quarterly and any entry untouched for two review cycles gets flagged for pruning; and nothing from a subagent's intermediate reasoning crosses back to the orchestrator, only its final structured result does. Write that down once, as an actual policy, and every engineer touching the system inherits the same admission and trust rules instead of reinventing them ad hoc per feature.

This is also why CDG deserves to be versioned and reviewed the way any other governance artifact would be — a data retention policy, an access-control list — rather than living as tribal knowledge in one architect's head. As the rest of this module will show, nearly every failure mode covered here (error propagation, ambiguity mishandled, provenance silently dropped in a summary) is really a CDG policy that was either never written down or never enforced.

<!-- prevnext:start -->

---

| [&larr; Previous: Managing the Context](02-managing-the-context) | [Next: Dealing with Escalation and Ambiguity &rarr;](04-dealing-with-escalation-and-ambiguity) |
|:---|---:|

<!-- prevnext:end -->
