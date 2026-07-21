---
title: Granularity on Options... Includes The Option To Not Do Something
parent: The Context Layer
nav_order: 5
duration: "0:40"
---

# Granularity on Options... Includes The Option To Not Do Something
{: .no_toc }

*Video duration: 0:40*

When you design the set of options an agent can choose from at a decision point, it's easy to only list the active ones: approve, reject, escalate, retry. That framing quietly forces a choice even when the honest answer is "I don't have enough information to pick any of these yet." Granularity in option design has to include an explicit **do-nothing-and-ask** option alongside the active ones — not as a last resort buried in the system prompt, but as a first-class choice the agent is told is always acceptable. Without it, an uncertain agent doesn't stop; it picks the least-bad active option and calls that a decision.

{: .important }
> If the option set handed to an agent never includes an explicit **do-nothing-and-ask** choice, an uncertain agent won't stop — it will pick the least-bad active option and present that as a decision. Always check that restraint is a first-class option, not something the agent has to invent by refusing its instructions.

This matters more than it sounds, because language models have a well-documented bias toward producing *an* answer over admitting insufficient grounds for one — the training pressure toward being helpful and decisive doesn't automatically distinguish "decisive because the evidence supports it" from "decisive because stopping felt like failing to help." If the option set handed to the model is `{approve, reject, escalate}`, an agent facing a genuinely ambiguous case will often force itself into the option that seems least harmful, rather than surfacing that none of the three actually fit. That's not a reasoning failure so much as a design failure: the option set itself never made room for the honest answer.

A concrete example: an expense-approval agent reviewing a submitted receipt sees an amount that's just over the category limit, with a note that doesn't clearly explain why. A three-option set — approve, reject, escalate to a human — pressures the model toward escalate, which sounds safe but silently trains the system to escalate anything mildly unclear, defeating the point of automating the easy cases. A four-option set that adds **ask the submitter a clarifying question and wait** lets the agent resolve the ambiguity cheaply, at the source, rather than dumping an under-specified case on a human reviewer who has no more information than the agent did.

This connects directly to the failure-affordance idea elsewhere in this module: how costly it is to pause and ask varies by context, so the do-nothing option isn't free everywhere, but it should always at least be *considered* and explicitly ruled out or in, rather than never existing as a candidate. Designing the option set is itself a governance decision — deciding what's admissible as a "decision" is a smaller instance of the Context Data Governance discipline named earlier in this module, applied to actions rather than data.

<!-- prevnext:start -->

---

| [&larr; Previous: Dealing with Escalation and Ambiguity](04-dealing-with-escalation-and-ambiguity) | [Next: Preventing Error Propagation &rarr;](06-preventing-error-propagation) |
|:---|---:|

<!-- prevnext:end -->
