---
title: Failure Affordance Varies By Use Case
parent: The Context Layer
nav_order: 11
duration: "0:34"
---

# Failure Affordance Varies By Use Case
{: .no_toc }

*Video duration: 0:34*

How much autonomy — and how much risk of being wrong — a system can tolerate is not a fixed property of "agents" in general; it varies by what's actually at stake if the agent gets it wrong. An agent drafting an email a human will read before sending can afford real guessing and even outright mistakes, because catching and fixing an error is cheap. An agent initiating an irreversible financial transaction can afford almost none. Every technique in this module — escalation thresholds, checkpoint placement, re-verification — should be calibrated to that use case's actual failure affordance, not applied uniformly everywhere.

The variable that actually drives failure affordance is **reversibility**, not the surface complexity or stakes-sounding of the task. A mistake in a drafted email is fully reversible before it's sent — a human catches it, edits it, nothing downstream ever happened. A mistake in a database query that only reads data is reversible in the trivial sense that nothing changed. A mistake in a database migration that writes and drops columns, or a wire transfer that clears in seconds, is not reversible in any practical sense once it executes — there's no "undo," only a costly cleanup, or no cleanup at all. Two tasks that look similarly "important" on the surface can sit at opposite ends of this spectrum: drafting a legal contract clause (reversible — a lawyer reviews it before it's signed) versus auto-renewing that same contract based on an inferred deadline (not reversible if the inference was wrong and the renewal window has passed).

{: .important }
> The variable that determines how much autonomy a system can tolerate is **reversibility, not how important or complex the task looks on the surface**. Two similarly "important"-looking tasks can sit at opposite ends of the failure-affordance spectrum — calibrate escalation thresholds, checkpoints, and re-verification to reversibility, not to surface stakes.

This has a direct, practical implication for how you tune the mechanisms covered elsewhere in this module. High failure-affordance systems can run with generous escalation thresholds, infrequent checkpoints, and lighter re-verification, because the cost of an occasional miss is low and cheaply corrected — over-engineering safety here just adds friction and latency for no real benefit. Low failure-affordance systems need the opposite: tight escalation thresholds, checkpoints before anything irreversible executes, and re-verification of any load-bearing fact rather than trusting a single unconfirmed source. Applying the low-affordance system's caution to the high-affordance case wastes effort and slows a system that didn't need the protection; applying the high-affordance system's looseness to the low-affordance case is how a plausible-sounding but wrong guess turns into an unrecoverable outcome.

The practical exam-relevant takeaway: when a scenario question describes an agent's autonomy level, don't evaluate whether it's "too autonomous" or "too cautious" in the abstract — evaluate it against what happens if this specific agent, doing this specific task, gets it wrong. The right amount of caution is a function of the blast radius, not a universal setting to be dialed the same way for every agent in a system.

<!-- prevnext:start -->

---

| [&larr; Previous: Designing Human Review](10-designing-human-review) | [Next: Data Provenance and Conflicts &rarr;](12-data-provenance-and-conflicts) |
|:---|---:|

<!-- prevnext:end -->
