---
title: Useful Metaphors and Analogies
parent: Additional Topics
nav_order: 7
duration: "1:20"
---

# Useful Metaphors and Analogies
{: .no_toc }

*Video duration: 1:20*

None of these prove anything, but they're sticky, and sticky beats precise when recalling a distinction under exam pressure.

- **Agentic layer = a web server's request loop.** Receive context, handle one request (respond or call a tool), send the result back, wait for the next request. An agent that "goes rogue" is a loop with a badly configured handler, not a personality having a bad day.
- **Tool layer = API design, except the caller is a model reading prose.** You're not writing docs for a developer who can search your codebase — you're writing the entire interface a reasoning system has to disambiguate from, cold.
- **Workflow layer = the org chart plus the runbook.** `CLAUDE.md` is the onboarding doc every hire reads; a Skill is the runbook pulled out for one specific job; a fixed pipeline is the checklist nobody deviates from.
- **Prompt layer = a contract, not a conversation opener.** It should hold up the same way every time it's invoked, which means it gets versioned, tested, and reviewed like code.
- **Context layer = the whiteboard everyone's writing on.** Finite space, shared visibility — whatever's crowding the board is exactly what gets reasoned over next, nothing more.
- **Subagent = a specialist you hire for one job, not a new employee.** They get a clean desk (fresh context), a specific brief, and they hand back a report — not a running commentary of everything they did to produce it. If the report is longer than the brief, something's wrong with the scoping.
- **The do-nothing option = a fire alarm's manual pull, not a bug.** A system that can only ever take an active step, never explicitly stop and flag uncertainty, isn't more capable than one with a "pause and ask" option — it's just louder about its mistakes.
- **Provenance = a chain-of-custody tag, not a footnote.** Data without a "where did this come from, and when" label isn't information the system can trust or distrust appropriately — it's just an assertion floating unanchored in context, indistinguishable from any other assertion.

None of these metaphors survive rigorous interrogation — push on any of them hard enough and they break down, the way all analogies eventually do. That's fine; they're not meant to be the thing you cite in an exam answer, they're meant to be the thing that fires first in your head when you read a scenario question, fast enough to point you toward the right layer before you've consciously started reasoning about it. Use the precise vocabulary from the five deep-dive modules to actually justify an answer, and use these to get there quickly in the first place.
