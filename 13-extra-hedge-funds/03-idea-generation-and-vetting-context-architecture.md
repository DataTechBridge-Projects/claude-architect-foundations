---
title: Idea Generation/Vetting - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Hedge Funds"
nav_order: 3
duration: "7:28"
---

# Idea Generation/Vetting - Context Architecture
{: .no_toc }

*Video duration: 7:28*

Once the agentic pipeline is settled — screen, generate, vet, each as its own agent — the next question is what actually goes into each agent's context window, because a pipeline of well-separated agents can still fail if each stage is fed the wrong slice of information. The temptation in a data-rich domain like finance is to over-provide: dump the whole filing, the whole price history, the whole thesis archive into context "just in case." That instinct is exactly backwards for this use case. Every extra token of irrelevant market data is a token competing for the model's attention against the two or three facts that actually matter, and in a domain where a model's fluency can outrun its grounding, a crowded context window makes it easier for confident-sounding but ungrounded reasoning to slip through unchallenged.

## Three Categories of Context

The context this system needs breaks cleanly into three categories, and each one is scoped and refreshed differently.

**Market and company data** is the most time-sensitive category: current price and volume, recent earnings figures versus consensus, relevant peer comparisons, and the specific filing or transcript excerpt that triggered the screening flag. This data has a short shelf life by design — a price quote or an earnings estimate that's even a day stale can change whether a thesis is still live. It's pulled fresh at the point of use rather than cached in any long-lived project configuration, and every fact from this category that ends up in a drafted thesis carries an explicit timestamp so a downstream reader (human or agent) can judge how current it is.

**Past thesis history** is the fund's own institutional memory: prior theses on the same name or sector, how they played out, and — critically — theses that were generated and rejected before, along with the reason they were rejected. This category is scoped narrowly to relevance rather than pulled in bulk; a screening flag on a semiconductor name should retrieve the fund's semiconductor thesis history, not its entire multi-year archive across all sectors. Its purpose is less about it being time-sensitive and more about it being the mechanism that prevents the system from re-proposing an idea the desk already tried and abandoned, or from repeating a mistake with a slightly different label. It refreshes on every new candidate, since the relevant slice of history depends entirely on what the candidate is about.

**Risk factor definitions** are the most stable category in the whole system — the fund's own taxonomy of the factors it tracks exposure against (sector concentration, rate sensitivity, liquidity, correlation to existing positions, and so on) rarely changes week to week. Because it's stable and relatively small, this is a reasonable candidate for something closer to persistent, always-loaded context for the vetting agent specifically — analogous to how a `CLAUDE.md` file loads standing conventions rather than being re-retrieved every time. It still needs a refresh discipline, though: if the fund revises its risk taxonomy, every agent instance has to pick up the new definitions immediately, not on some lagging cache-invalidation schedule, because vetting against a stale risk taxonomy is a silent, hard-to-detect failure mode.

## Scoping Context Per Agent, Not Per System

Because the three agents in this pipeline have different jobs, they don't get the same context — this is where context architecture and agentic architecture reinforce each other directly. The screening agent needs breadth and freshness: a wide but shallow slice of recent filings, transcripts, and news, without the past thesis archive or the risk taxonomy, because its job is pattern-spotting, not judgment. Loading it down with the fund's full risk-factor definitions would only slow it down and risk it second-guessing candidates it should simply be flagging for a later stage to judge.

The generation agent needs depth on one candidate: the specific filing or transcript that triggered the flag, closely relevant peer and market data, and enough past thesis history on that name or sector to avoid reinventing an idea that already failed. It does not need the full risk-factor taxonomy loaded either — that's deliberately withheld, because the point of separating generation from vetting is that the generation agent should build the most complete, honest version of the thesis it can without shaping its narrative around what it thinks the risk desk wants to hear.

The vetting agent gets the opposite mix: the drafted thesis itself, the full risk-factor taxonomy, targeted retrieval aimed specifically at finding *contradicting* evidence (a different retrieval objective than the generation agent's, discussed in the next lecture), and the same past-thesis-history slice so it can check whether this exact idea failed before. Giving the vetting agent the risk taxonomy while withholding it from the generation agent is a direct, deliberate application of context isolation — it's the same principle as keeping one subagent's noisy intermediate reasoning from leaking into another's context, applied here to keep one agent's evaluative framework from leaking into another agent's generative framework.

| Context Category | Freshness Discipline | Screening Agent | Generation Agent | Vetting Agent |
|---|---|---|---|---|
| Market & company data | Pulled fresh at point of use; timestamped | Wide, shallow slice | Deep, candidate-specific slice | Same slice as generation |
| Past thesis history | Refreshed per candidate; scoped to relevance | Not loaded | Narrow slice on that name/sector | Same narrow slice |
| Risk factor definitions | Stable; refreshed immediately on change | Not loaded | Withheld deliberately | Full taxonomy, always loaded |

## Freshness and Refresh Discipline

Because market conditions move faster than any static configuration file could track, this system treats "when was this fact retrieved" as a first-class piece of context, not an afterthought. Every piece of market or company data entering context is timestamped at retrieval time, and both the generation and vetting agents are instructed to treat data older than a defined threshold (say, more intraday-sensitive data older than a few hours, or fundamentals older than the most recent filing) as stale and to say so explicitly rather than silently reasoning over it as if it were current. This is a provenance discipline as much as a freshness one: a thesis that cites a data point is expected to carry that data point's source and recency along with it, so a human reviewer isn't left guessing whether "revenue grew 12%" refers to this quarter or a stale cached figure from three cycles ago.

The risk-factor taxonomy sits at the other end of the freshness spectrum and is refreshed on change rather than on a timer — there's no value in re-fetching stable definitions on every run, but there's real danger in an agent vetting against a taxonomy version that's been superseded. The practical implementation is a version-tagged taxonomy source: every vetting agent invocation records which taxonomy version it evaluated against, so if the taxonomy changes, previously vetted (and rejected) ideas can be identified for re-evaluation rather than silently left stale in the archive.

## Architecture Notes

This lecture is a direct application of the Context layer's central discipline: context isn't one shared pool, it's something scoped deliberately per agent and per purpose, with freshness and provenance tracked as explicitly as the content itself. Withholding the risk taxonomy from the generation agent while loading it for the vetting agent is context isolation in service of the Agentic layer's generator/vetter split — a reminder that context and agentic design are two views of the same underlying decision about what each piece of the system is allowed to know and when.

<!-- prevnext:start -->

---

| [&larr; Previous: Idea Generation/Vetting - Agentic Architecture](02-idea-generation-and-vetting-agentic-architecture) | [Next: Idea Generation/Vetting - RAG.& Retrieval Architecture &rarr;](04-idea-generation-and-vetting-rag-and-retrieval-architecture) |
|:---|---:|

<!-- prevnext:end -->
