---
title: Role Play: Mentoring on Context Essentials
parent: The Context Layer
nav_order: 18
---

# Role Play: Mentoring on Context Essentials

**Mentee:** I've got an agent triaging IT support tickets — password resets, access requests, that kind of thing. It's been running two weeks and yesterday it reset the wrong person's password. Same first and last name, different department. I want to just add a rule: "if names match, always ask." Simple fix, right?

**Mentor:** That's a workflow patch, and it might even help. But before we bolt on a rule, let's look at what was actually in its context when it made the call. What did it see?

**Mentee:** The ticket said "reset password for John Kim," and it looked up John Kim in the directory. There was only one John Kim returned by the lookup it ran, so it went ahead.

**Mentor:** So the lookup wasn't ambiguous from the agent's point of view — it got exactly one match and treated that as certainty. Where's the actual ambiguity, then?

**Mentee:** ...There were two John Kims in the company. Its directory tool just happened to only return the one in a certain department by default, some filter I didn't know was on.

**Mentor:** So the ambiguity was real, but it was invisible to the agent, because the tool result made a genuinely uncertain situation look like a clean, single-answer lookup. That's not a "did the agent guess" problem — it's a provenance problem. The tool result had no marker saying "this is one of possibly several matches, filtered by department." It just looked like ground truth.

**Mentee:** So the fix isn't "always ask when names match" — it's making sure the agent can even tell when a match might not be unique.

**Mentor:** Right. Think about what we covered on data provenance. Every fact in context needs a sense of how it was obtained and how much confidence it deserves. A directory lookup that silently applies a filter and returns one result is handing the agent a false sense of certainty. Fix the tool so it either returns all plausible matches or flags when a filter narrowed the result, and now the agent has the information it needs to recognize the ambiguity itself.

**Mentee:** And once it recognizes it, what should it actually do? Just stop and escalate the whole ticket?

**Mentor:** Depends on the granularity question. What's the actual decision at stake?

**Mentee:** Which account's password to reset.

**Mentor:** And how reversible is getting that wrong?

**Mentee:** Not very — the wrong person suddenly can't log in, and the actual John Kim who needed it still can't either. That's what happened yesterday.

**Mentor:** So this isn't a "flag an assumption and proceed" situation — the cost of being wrong is real and immediate. But it's also not a "stop the entire ticket queue" situation. It's one specific decision point. What's the narrowest response that resolves it?

**Mentee:** Ask which John Kim — give the department or employee ID as the disambiguator — before doing the reset. One clarifying question, not a full escalation.

**Mentor:** Exactly. And notice what made that answer available to you: not a blanket "always ask" rule, but the tool surfacing its own uncertainty, plus an agent designed with "ask a narrow clarifying question" as a real option alongside "just reset it." If that option isn't built in, the agent only has "reset" or "don't reset" to choose from, and an uncertain case gets forced into one of those two whether it belongs there or not.

**Mentee:** So the actual fix is two layers down from where I was looking — it's the tool's provenance, not the agent's decision rule.

**Mentor:** That's usually where these bugs live. By the time you're looking at the wrong decision, the context that produced it already lied to you a step earlier.
