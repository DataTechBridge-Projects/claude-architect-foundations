---
title: Trade Risk Management/Compliance - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Hedge Funds"
nav_order: 8
duration: "7:12"
---

# Trade Risk Management/Compliance - Context Architecture
{: .no_toc }

*Video duration: 7:12*

The compliance review agent's output is only as trustworthy as the context it's evaluating against, and in this domain "trustworthy context" has a sharper meaning than in most: it's not just about relevance, it's about being certain the agent is reasoning against the *current, correct* version of every rule, limit, and counterparty fact in play. Getting context architecture wrong here doesn't just produce a slightly worse answer — it can produce a confidently wrong compliance assessment that looks identical to a correct one until someone checks the underlying rule text by hand.

## The Three Core Context Inputs

**Position and exposure limits** are the most structured input and the one most amenable to being pulled as clean, current data rather than retrieved as prose — the agent needs the fund's actual current positions, its limit thresholds by strategy or desk, and how a proposed trade would change exposure if executed. This data comes from the fund's own position-keeping systems, not from any document store, and it needs to be current as of the moment of review, not a cached snapshot from earlier in the day — a limit check run against stale positions is one of the more dangerous failure modes in this system, since a trade that looked fine against morning positions might breach a limit against positions updated by trades that executed an hour ago.

**Regulatory rules** are the least structured and most consequential input: the actual text of the regulations, internal compliance policies, and restricted lists that apply to the fund's jurisdiction, strategy, and the specific instrument or counterparty involved. Unlike position data, this isn't something to be computed fresh each time — it's retrieved from a maintained document store, which is the subject of the next lecture on retrieval architecture. What matters at the context-architecture level is scoping: a review of a single-name equity trade doesn't need the fund's entire compliance manual in context, it needs the subset of rules that actually apply to that instrument type, that jurisdiction, and that counterparty — a much smaller, much more focused slice that both keeps the context window usable and reduces the chance the agent conflates a rule that doesn't apply with one that does.

**Counterparty data** — who the fund is trading with, that counterparty's regulatory status, any flags or restrictions associated with them, and relevant history of past dealings — sits in between the other two in volatility. It changes far less often than positions but more often than most regulatory text, and it needs its own refresh discipline: counterparty status (sanctions exposure, heightened-scrutiny flags, credit concerns) should be checked against a source that's updated at least daily, because a counterparty's status can change for reasons entirely outside the fund's own trading activity — a sanctions designation or a credit downgrade that happened overnight has to be reflected before the next trade involving that counterparty is reviewed, not caught retroactively.

| Context Input | Source | Refresh Cadence | Volatility |
|---|---|---|---|
| Position & exposure limits | Fund's position-keeping systems | Real time, as of the moment of review | Highest — can shift intraday |
| Regulatory rules | Maintained document store | Refreshed on change; scoped per trade | Lowest — stable but consequential |
| Counterparty data | Compliance / reference data source | At least daily | Moderate |

## Scoping Context to the Specific Trade

Just as the idea generation system scoped context per agent, this system scopes context per *trade*, and tightly. A proposed trade's review context should include only the limits that apply to its specific strategy and desk, only the regulatory rules that apply to its instrument type and jurisdiction, and only the counterparty data for the specific counterparty involved — not the fund's entire limit schedule, not its entire compliance manual, not every counterparty on file. This isn't only an efficiency concern, though it is that; it's a correctness concern, because a wider context window gives the agent more opportunity to apply a rule that sounds relevant but technically doesn't govern this trade, or to miss the one rule that does because it's buried among dozens of others that don't apply. Precise scoping is itself a risk-reduction technique in this domain, not just a performance optimization.

## Handling Conflicting or Stale Rules

Compliance rules genuinely conflict sometimes — an internal policy might be more conservative than the minimum required by regulation, two jurisdictions' rules might point in different directions for a cross-border trade, or a newly issued rule might not yet have a correspondingly updated internal policy to match it. The context architecture has to make these conflicts *visible* rather than silently resolved by whichever rule the retrieval system happened to surface first or most prominently. Every rule loaded into the agent's context carries explicit **provenance**: its source (regulation, internal policy, restricted list), its effective date, and its version — and when two applicable rules genuinely conflict, the agent is instructed to surface the conflict explicitly in its output rather than pick one silently, flagging it as exactly the kind of ambiguity that should route to human escalation rather than be resolved by model judgment.

Staleness is handled with the same discipline applied to market data in the idea generation use case, but with much less tolerance for ambiguity about what counts as current. Every rule and every piece of counterparty data carries a last-verified timestamp, and the agent is required to treat data past a defined staleness threshold as unusable rather than degraded-but-usable — where the idea generation system might flag a stale market data point and still reason around it, this system's default on stale compliance data is to halt and request a refresh, because reasoning around potentially outdated regulatory text is precisely the kind of shortcut this domain cannot tolerate. This ties directly to provenance: a rule without a clear, verifiable source and effective date is treated as insufficient basis for any assessment, full stop, regardless of how confidently it reads.

## Architecture Notes

Tight per-trade scoping and mandatory provenance on every rule are the Context layer's core disciplines applied at maximum strictness, because in this domain an ungrounded or stale context item isn't just lower-quality output, it's a compliance failure. Surfacing rule conflicts explicitly rather than silently resolving them is the same escalation principle from the Agentic layer expressed at the context level: when context itself is ambiguous or contradictory, the system's job is to make that visible to a human, not to quietly pick a winner.

<!-- prevnext:start -->

---

| [&larr; Previous: Trade Risk Management/Compliance - Agentic Architecture](07-trade-risk-management-and-compliance-agentic-architecture) | [Next: Trade Risk Management/Compliance - RAG & Retrieval Architecture &rarr;](09-trade-risk-management-and-compliance-rag-and-retrieval-archi) |
|:---|---:|

<!-- prevnext:end -->
