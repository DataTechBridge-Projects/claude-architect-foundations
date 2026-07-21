---
title: Conway's Law for Multi-Pass Execution?
parent: The Prompt Layer
nav_order: 13
duration: "0:49"
---

# Conway's Law for Multi-Pass Execution?
{: .no_toc }

*Read duration: 0:49*

Conway's Law observes that organizations design systems that mirror their own communication structure — the software ends up shaped like the org chart that built it. Multi-pass review pipelines have their own version of this. If your organization already separates "the people who draft" from "the people who approve," your generation and review prompts will naturally fall along that same seam, and the review pass will inherit whatever authority, blind spots, or incentives that team already has in the real process. **Before assuming a review pass is independent because it's architecturally separate, check whether it's actually independent in judgment** — or whether it's just reproducing the same division of responsibility, and the same gaps, that already exist around it organizationally.

## A Concrete Example

Picture a content moderation system where a "drafting" prompt decides whether a post is safe to publish, and a "review" prompt double-checks that decision before it goes live. If both prompts were written by the same two engineers, tuned against the same set of example posts, and structured around the same taxonomy of violation categories, the review pass isn't actually testing the drafting pass's judgment against an independent standard — it's testing whether the same judgment, applied twice, agrees with itself. Both prompts will share the same blind spots, because they were built by people who share the same assumptions about what "safe" means and the same gaps in the example set they tuned against. A genuinely novel violation type that neither engineer anticipated will sail through both passes, not because the second pass failed, but because it was never actually independent in the way the architecture diagram suggests.

## Designing for Real Independence

Breaking this pattern takes deliberate effort, because the default gravitational pull — per Conway's Law — is toward mirroring existing structure. A few concrete levers help: give the review pass a genuinely different objective ("actively look for reasons this should NOT have been approved" rather than "confirm this looks fine"), source its criteria or examples from a different team or a different incident history than the drafting pass used, and consider using a different model, temperature, or prompting strategy so the review pass isn't just re-deriving the same reasoning path the draft pass already took. An adversarial framing — instructing the review pass to actively try to find a reason to reject, rather than to passively confirm — is one of the more reliable ways to break the mirrored-judgment trap, because it forces a different cognitive stance rather than just a second pass through the same stance.

**Why it matters for the exam:** Expect a scenario describing a two-pass review pipeline that still lets a known failure mode through, and asking why independence didn't help. The likely answer is that the "independent" review pass shared the same design assumptions, criteria, or team origin as the drafting pass — real independence requires deliberately different criteria or perspective, not just a second prompt in the pipeline.

<!-- prevnext:start -->

---

| [&larr; Previous: Independent & Multi-Pass Review](12-independent-and-multi-pass-review) | [Next: Lab: Prompt Reliability Simulator &rarr;](14-lab-prompt-reliability-simulator) |
|:---|---:|

<!-- prevnext:end -->
