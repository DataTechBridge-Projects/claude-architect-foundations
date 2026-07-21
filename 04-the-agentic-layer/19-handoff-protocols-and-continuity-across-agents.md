---
title: Handoff Protocols and Continuity Across Agents
parent: The Agentic Layer
nav_order: 19
duration: "11:00"
---

# Handoff Protocols and Continuity Across Agents
{: .no_toc }

*Read duration: 11:00*

The previous lecture treated the agent-to-agent message as a schema — a task description, curated context, an expected output shape. This one treats a narrower but related question: when work genuinely moves from one agent (or one session) to another mid-task, what does the receiving side actually need to pick up correctly, and how much of that need is really about *continuity* rather than instruction? A handoff isn't just a dispatch — it's a transfer of an unfinished task's state, and getting that transfer right means answering a question that sounds simple and isn't: how much of what came before does the next agent actually need to see?

## The instinct to over-share, and why it fails

The default instinct, when handing off mid-task work, is to be maximally safe by including everything — the full upstream conversation, every tool call and result, every dead end explored along the way — on the theory that more context can only help. This is exactly the instinct the "Results Polluting the Context" aside warned against at the level of a single tool result, and it's just as wrong applied at the level of a full handoff, for the same underlying reason: a receiving agent's context is a scarce, finite resource, and every token spent re-reading exploration that's already resolved is a token not available for reasoning about what's actually left to do.

The costs compound in a handoff specifically because handoffs tend to happen at exactly the moments when the upstream context is largest — after a long investigation, deep into a multi-step task, following several rounds of trial and error. Dumping that raw history into the next agent doesn't just cost tokens; it actively degrades the receiving agent's reasoning, because it now has to sift signal from a transcript full of resolved dead ends, superseded plans, and intermediate noise that was only ever useful to the agent that produced it in the moment. A handoff designed this way looks generous but behaves like the noisiest possible tool result, carried forward at the scale of an entire task rather than one call.

## What a receiving agent actually needs

The alternative isn't withholding information out of stinginess — it's being deliberate about what a handoff is actually for. A receiving agent needs enough to continue the task *correctly*, not enough to reconstruct everything the prior agent ever considered. In practice, that's a short, specific list:

- **The current state of the task** — what's been completed, what remains, phrased as a concrete status rather than a narrative of how it got there. "Draft written and fact-checked; two citations still need sourcing" tells the next agent exactly what to do next. A full transcript of the drafting and fact-checking process doesn't add anything actionable on top of that summary — it just makes the actionable part harder to find.
- **Decisions already made and why**, specifically the ones that constrain what the receiving agent is allowed to do next. If an earlier agent decided against a particular approach for a real reason — a data source that turned out to be unreliable, a constraint the user stated explicitly — that decision and its reason need to survive the handoff, or the receiving agent risks re-deciding it from scratch, potentially differently, and re-doing work that was already settled.
- **Open questions or blockers**, named explicitly rather than left to be rediscovered. A task handed off mid-investigation because of an unresolved ambiguity should say so directly, rather than relying on the receiving agent to notice the ambiguity itself buried somewhere in a longer history.
- **Anything the receiving agent cannot cheaply rediscover on its own** — a specific file path, an external identifier, a result from a tool call that would be expensive or slow to repeat. This is the genuine exception to minimalism: information that's cheap to include but expensive to regenerate belongs in the handoff even if it isn't strictly "decisions" or "status," because the alternative is the receiving agent burning its own loop rediscovering something the prior agent already had in hand.

Notice the shape of this list: it's a *distillation* of the upstream work, not a redaction of it. The discipline is the same one subagent report design asks for — separate the conclusion and its load-bearing support from the process that produced it, and hand forward the former, not the latter.

{: .important }
> A handoff is a curated distillation of state, decisions, and open questions — not a completeness guarantee achieved by including everything. Dumping the full upstream transcript "to be safe" degrades the receiving agent's reasoning rather than protecting it.

| Handoff component | Why it matters |
|---|---|
| Current state of the task | Tells the next agent exactly what to do next, without a narrative to sift through |
| Decisions already made and why | Prevents re-deciding, possibly differently, work that was already settled |
| Open questions or blockers | Surfaces ambiguity directly instead of leaving it to be rediscovered |
| Anything not cheaply rediscoverable | Saves the receiving agent from re-doing expensive or slow work |

## Session and state continuity across a handoff

A handoff carries a second kind of continuity beyond task content: whatever session-level state the task depends on. Session management established that a session is the accumulating record — transcript, working state — that makes multi-turn interaction coherent over time, and a handoff mid-task means that record's ownership is changing hands, which raises questions a same-session continuation never has to answer.

If the receiving agent operates in its own fresh session — the common case for a subagent, or for handing a task to a genuinely separate system — the handoff payload is, in effect, the *entire* initial context that agent will have, in exactly the way subagent instantiation described: nothing carries over implicitly, and anything not explicitly included in the handoff is gone from the receiving agent's point of view, regardless of how present it was in the sending agent's own session. This is why the distillation discipline above matters more, not less, at a genuine session boundary — there's no ambient shared history to fall back on if the handoff turns out to have omitted something.

If instead the receiving agent resumes within the same underlying session — a task picked back up later in one long-running interaction — continuity is partly free, in that the session's own history is still technically present. But session management's own lesson applies here too: a session that's grown long enough to need a distilled handoff in the first place is exactly the kind of session where unmanaged history growth has probably already made "just look back at the transcript" an unreliable way to resume, since the relevant few decisions are diluted among everything else that's accumulated. A well-designed handoff, in this case, functions as a deliberate checkpoint summary — the same kind of artifact session management's own summarization strategy produces — inserted specifically at the point where responsibility changes hands, rather than trusting the receiving agent to correctly re-derive "what actually matters right now" from raw history alone.

## A worked example

Consider a long research task that starts with one agent gathering sources on a technical topic across dozens of searches and page reads, then hands off to a second agent whose job is to write the final report. A poorly designed handoff passes the entire search-and-read transcript forward — every query tried, every page skimmed and discarded, every dead end — on the assumption that the writing agent might need any of it. The writing agent now has to spend a meaningful chunk of its own context just figuring out which of the dozens of sources actually matter, effectively redoing the first agent's own filtering work, except with less clarity about why any given source was kept or discarded, since that reasoning lived in the first agent's process rather than in anything explicitly carried forward.

A well-designed handoff instead passes: the distilled list of sources judged relevant, each with a one-line note on why it matters and what claim it supports; any sources deliberately excluded and why, if that reasoning is load-bearing (a source that looked relevant but turned out to be outdated, say, which the second agent should not re-discover and re-include); and an explicit statement of what's still missing, if the first agent's research was cut off before feeling complete. The writing agent, given this, starts its own work from a clean, current, and complete-enough state — able to write, cite, and flag gaps immediately, rather than spending its own budget reconstructing a filtering judgment the first agent had already made and simply failed to hand forward explicitly.

## Why it matters for the exam

Expect the exam to describe a handoff — sometimes a lean, well-designed one, sometimes one that dumps an entire upstream transcript — and ask you to identify what the receiving agent actually needed versus what it was needlessly given, or to identify a genuine gap where something load-bearing was left out. The core distinction to hold onto is that a good handoff is a curated distillation of state, decisions, and open questions, not a completeness guarantee achieved by including everything — and that this is the same context-discipline principle from results pollution and subagent reporting, applied specifically to the moment responsibility for a task changes hands.

<!-- prevnext:start -->

---

| [&larr; Previous: Designing Agent-to-Agent Message Schemas](18-designing-agent-to-agent-message-schemas) | [Next: Retry, Fallback, and Escalation Trigger Design &rarr;](20-retry-fallback-and-escalation-trigger-design) |
|:---|---:|

<!-- prevnext:end -->
