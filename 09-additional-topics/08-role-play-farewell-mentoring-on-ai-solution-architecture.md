---
title: "Role Play: Farewell Mentoring on AI Solution Architecture"
parent: Additional Topics
nav_order: 8
---

# Role Play: Farewell Mentoring on AI Solution Architecture

**Mentee:** Before I go sit the exam, can I run the last design by you? It's the capstone scenario from the practice set: a Claude-based system that reviews vendor contracts for a legal ops team and flags clauses that deviate from company standard.

**Mentor:** Go. Start wherever you want — you pick the layer this time.

**Mentee:** Agentic first, since that's usually where I go wrong. I want a single agent, not a fixed workflow, because "does this clause deviate from standard" genuinely needs judgment — you can't enumerate every phrasing of an indemnification clause in advance. But not multi-agent, because there's no real specialization split here, just one review task repeated per clause.

**Mentor:** Good — and you named the unpredictability instead of just asserting it, which is exactly the discipline that pattern is testing. Keep going.

**Mentee:** Tool layer: three tools. Look up the company's standard clause library, look up this vendor's contract history, and flag a clause with a severity and explanation — deliberately no "send to counterparty" or "auto-reject" tool. The highest-stakes actions stay outside what the agent can do at all.

**Mentor:** Why flag with a severity instead of just a boolean deviates/doesn't-deviate?

**Mentee:** Because a boolean forces every deviation into the same bucket, and a one-word typo variance isn't the same risk as a missing liability cap. Structure where I can validate it — severity as an enum — prose for the explanation, since that's genuinely judgment-dependent.

**Mentor:** Structure over prose, prose as the escape hatch. Workflow layer?

**Mentee:** Flagged clauses above a severity threshold go through a human review gate before anything moves forward — that's not optional, that's the irreversible-action boundary, since a bad clause getting through is expensive to unwind after signature. Below threshold, it logs and continues; I'm not gating everything, just where the cost of being wrong is real.

**Mentor:** And the gate itself — what does the reviewer actually see?

**Mentee:** Just the flagged clause, the matched standard clause it's compared against, and the stated reason — not the whole contract, not the whole session. A gate that dumps everything on the reviewer doesn't function, it just exists.

**Mentor:** That's the distinction the whole Context layer kept circling back to. Prompt and context layers — close it out.

**Mentee:** Prompt: a handful of examples spanning the real range — a subtle deviation, an obvious one, a false positive that looks like a deviation but isn't — not three examples that all look alike. And the flag-writing instruction is explicit about severity criteria, not left for the model to infer consistently on its own. Context: only this contract and the relevant slice of the standard clause library get loaded, not the whole library, and every clause pulled in carries where it came from and when it was last updated — so if the standard itself changed last quarter, the agent isn't comparing against something stale without knowing it.

**Mentor:** That's a genuinely sound design, and notice what you just did — you made five independent decisions, each defensible in its own layer's vocabulary, and none of them contradicted another. That's the whole exam in one scenario.

**Mentee:** So... that's it? I'm ready?

**Mentor:** As ready as a conversation can make you. On the exam itself: when a question feels ambiguous, ask which of the five domains it's actually testing before you touch the answer choices — you just did that instinctively for six layers in a row without naming it once. When two answers both technically work, take the simpler one. And when you're not sure, trace it back to the layer that actually produced the symptom, not the layer where you happened to notice it.

**Mentee:** And if I get a question I just don't know?

**Mentor:** Then you reason from the five questions each layer asks — who decides, what can it do, how fixed is the process, how is intent communicated, what does it actually see — and you'll get further than guessing ever would. You've done the work. Go show it.

<!-- prevnext:start -->

---

| [&larr; Previous: Useful Metaphors and Analogies](07-useful-metaphors-and-analogies) | [Next: Practice Exam &rarr;](../10-practice-exam/) |
|:---|---:|

<!-- prevnext:end -->
