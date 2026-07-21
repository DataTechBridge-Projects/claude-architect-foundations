---
title: Think of Scaling
parent: The Agentic Layer
nav_order: 7
duration: "0:29"
---

# Think of Scaling
{: .no_toc }

*Video duration: 0:29*

Going from one agent to many doesn't scale linearly — it scales worse. Cost multiplies: each subagent is its own set of billed model calls, on top of the orchestrator's. Rate limits bind harder: five parallel subagents can hit provider concurrency or throughput limits that a single agent never would. And coordination complexity grows non-linearly with the number of agents, since every added agent is a new potential point of failure, a new report to reconcile, and a new source of inconsistency with its siblings. Before fanning work out, ask whether the task actually needs it.

A worked example makes the multiplier concrete. Suppose a single agent doing a task directly costs roughly $0.50 in model calls. Fan that task out to five parallel subagents, each doing a comparable share of work, and you're not paying $0.50 total — you're paying something closer to five times that for the subagents' own calls, plus the orchestrator's overhead for dispatching each one and reconciling five separate reports, plus whatever duplicated setup (shared instructions, shared background context) gets re-sent into every subagent's starting prompt. If each subagent also independently retries on transient failures, or if two subagents happen to redo overlapping work because their scopes weren't cleanly partitioned, the real multiplier can run well past five. None of this shows up in a back-of-envelope estimate that just divides the total work by the number of agents.

Rate limits deserve a similar concrete framing. A provider's concurrency limit is typically set at the account or API-key level, not per agent — so five subagents launched at once aren't five independent budgets, they're five simultaneous draws against one shared ceiling. A design that works fine with one agent making calls at a steady pace can suddenly start throwing 429s the moment it's parallelized, not because any individual subagent is misbehaving, but because the aggregate concurrency crossed a limit that was invisible when there was only one caller.

This is why "think of scaling" belongs right after the multi-agent coordination lecture rather than standing alone: it's the blunt, memorable version of the coordination-overhead argument made there in full — latency overhead, cost overhead, reconciliation complexity, and failure attribution all get worse, not better, as agent count grows, and none of that is optional plumbing you can skip. The exam-relevant habit is to treat "how many agents" as a cost with its own justification, weighed against the specific benefit (parallelism, context isolation, specialization) a task actually needs, rather than a dial to turn up whenever a task feels big.

<!-- prevnext:start -->

---

| [&larr; Previous: Multi-Agent Coordination](06-multi-agent-coordination) | [Next: Subagent Instantiation &rarr;](08-subagent-instantiation) |
|:---|---:|

<!-- prevnext:end -->
