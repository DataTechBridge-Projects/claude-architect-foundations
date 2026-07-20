---
title: Should I Stay? Or Should I Propagate?
parent: The Context Layer
nav_order: 7
duration: "0:33"
---

# Should I Stay? Or Should I Propagate?
{: .no_toc }

*Video duration: 0:33*

Every uncertain fact an agent is holding faces the same fork the old song asks about, just rephrased: should this information stay right here and get resolved before moving on, or should it propagate forward into the next turn's context as if it were settled? "Stay" means pausing — verifying, escalating, or flagging the uncertainty explicitly so it travels labeled as uncertain if it travels at all. "Propagate" unlabeled is how a shaky guess in turn four becomes a load-bearing assumption in turn twenty. The default should be: if you're not sure, it stays and gets resolved — or it propagates loudly, never quietly.

Walk through a concrete version of the fork. In turn four of a multi-step research task, an agent infers that a company's headquarters moved to a new city, based on a document that mentions the city in passing without stating it as the current address. That inference is plausible but unverified. If it propagates silently — just written into the running summary as "the company is headquartered in [city]" — every later turn that reads the summary inherits it as settled fact, with no way to tell it was ever a guess. By turn twenty, an entirely different part of the task might be citing that city as grounding for its own conclusion, and the original uncertainty is now buried three layers deep in derived reasoning nobody thinks to re-examine.

The "stay" branch has real costs too, which is why it's a genuine decision and not an automatic default: pausing to verify an address might mean an extra tool call, an extra turn, or a clarifying question that slows the task down. That cost has to be weighed against how load-bearing the fact is likely to become — which is exactly the failure-affordance judgment covered elsewhere in this module. A minor detail that's unlikely to matter later can reasonably propagate loudly (tagged "unverified — inferred from passing mention") rather than blocking progress to confirm it immediately; a fact that later steps will treat as foundational deserves to stay and get resolved before anything is built on top of it.

The mechanism that makes "propagate loudly" actually work is the same provenance tagging covered later in this module: an uncertain fact that carries its uncertainty label with it through every summarization and compaction pass remains recognizably uncertain at turn twenty, even if the surrounding detail has long since been compressed away. Strip that label off during a careless summary, and loud propagation quietly becomes silent propagation — which is precisely the error-propagation failure mode this lecture's question is designed to prevent.
