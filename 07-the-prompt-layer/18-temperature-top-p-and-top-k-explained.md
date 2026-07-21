---
title: Temperature, top_p, and top_k Explained
parent: The Prompt Layer
nav_order: 18
duration: "10:00"
---

# Temperature, top_p, and top_k Explained
{: .no_toc }

*Video duration: 10:00*

Everything covered so far in this module shapes a model's output through the prompt — what it's told, what it's shown, what it's given to reason with. **Sampling parameters** are a different lever entirely: they control the mechanical process by which the model converts its internal probability distribution over possible next tokens into an actual chosen token, independent of what the prompt says. Three parameters — `temperature`, `top_p`, and `top_k` — govern that process on Claude's API, and understanding what each one actually does is what separates an architect who can deliberately tune a system's determinism from one who's guessing at a knob.

## The Underlying Mechanism

At every step of generation, the model produces a probability distribution over its entire vocabulary — for the next token, "the" might have a high probability, "a" a slightly lower one, and thousands of other tokens vanishingly small but nonzero probabilities. Left completely unconstrained, the model would need some rule for turning that distribution into a single chosen token, and the three parameters in this lecture are exactly that rule, applied in combination.

**Temperature** controls how sharply that distribution is reshaped before a token is sampled from it. A low temperature sharpens the distribution, concentrating probability mass onto the already-most-likely tokens and making the model's choices close to deterministic — at the extreme (temperature near zero), the model reliably picks the single highest-probability token every time, which is why very low temperatures are described as producing "deterministic" or "greedy" output. A high temperature flattens the distribution, giving lower-probability tokens a meaningfully higher chance of being selected, which produces more varied, more surprising, and occasionally more creative output — at the cost of also raising the odds of a genuinely poor or incoherent choice slipping through, since tokens that were unlikely for good reason are now getting selected more often too.

**top_p** (also called **nucleus sampling**) takes a different approach to the same distribution: rather than reshaping every probability, it restricts sampling to the smallest set of tokens whose cumulative probability reaches a threshold `p`, and samples only from within that restricted set. With `top_p` set to 0.9, the model considers only the most likely tokens that together account for 90% of the probability mass, discarding the long tail of unlikely tokens entirely before sampling. This adapts naturally to the shape of the distribution at each step — when the model is highly confident and one or two tokens dominate, the nucleus is small; when the model is genuinely uncertain and probability is spread across many plausible tokens, the nucleus widens to include more of them.

**top_k** is the simplest of the three: it restricts sampling to exactly the `k` most probable tokens, regardless of how much cumulative probability they represent, and discards everything outside that fixed-size set. Unlike `top_p`, which adapts its cutoff to the shape of the distribution, `top_k` always considers the same number of candidates, whether the model is highly confident (in which case a fixed `k` might include some genuinely implausible options) or highly uncertain (in which case a fixed `k` might exclude reasonable candidates that fell just outside the cutoff).

In practice these parameters combine: a request typically applies `top_k` and/or `top_p` to narrow the candidate set first, then applies temperature to reshape probabilities within what remains, before a token is finally sampled. An architect doesn't need to reimplement this pipeline, but understanding the order of operations clarifies why, for instance, a very restrictive `top_k` or `top_p` can make a high temperature setting largely irrelevant — if only one or two tokens survive the restriction, there's little for a flattened distribution to meaningfully vary across.

| Parameter | What It Restricts | Adapts to Distribution Shape? |
|---|---|---|
| Temperature | Reshapes the sharpness of the whole distribution | N/A — reshapes rather than restricts a set |
| top_p (nucleus) | Smallest set of tokens reaching cumulative probability `p` | Yes — the nucleus size adapts each step |
| top_k | A fixed number, `k`, of top candidates | No — always the same candidate count |

## Low vs. High Temperature in Practice

The practical decision an architect actually faces is rarely "what's the precise mathematical effect of top_p 0.9 versus 0.95" — it's "should this task be close to deterministic, or should it have room to vary?" Tasks that have a single correct or best answer — structured data extraction, classification, code generation against a precise specification, anything where two different valid-looking outputs for the same input would actually be a problem rather than a feature — call for low temperature, because the goal is consistency: the same input should reliably produce the same (or very nearly the same) output every time the system runs it, and unwanted variance here reads as a bug, not as creativity.

Tasks where variety is actually part of the value — generating several distinct marketing headline options, brainstorming names, writing varied creative copy across many similar requests where sameness would be dull or repetitive — benefit from higher temperature, because the entire point is to sample somewhat different, individually reasonable completions rather than converging on the single most probable one every time. The judgment call is really about whether variance in the output is a defect or a feature for the specific task at hand, and that's a question about the task, not about the model.

## Guidance for a Production Agentic System

For most of what this course treats as production agentic work — tool selection, structured data extraction, classification, code review, anything feeding a downstream system that expects consistent behavior — low temperature is the sound default, because reliability and reproducibility matter more than variety, and unpredictable variation in a tool-calling decision or an extracted field is a source of bugs, not a benefit. Reserve higher temperature deliberately for the narrower slice of tasks where output diversity is genuinely part of the goal, and treat it as an explicit, documented choice for that specific call site rather than a system-wide default.

It's worth noting that these parameters address *variance in token selection*, not correctness — a low-temperature setting makes a model's output more consistent, but consistently wrong is still wrong, and no combination of temperature, top_p, or top_k substitutes for the prompt-design and evaluation work covered throughout the rest of this module. Sampling parameters are a tuning layer applied on top of a well-designed prompt, not a replacement for one.

## A Worked Example

Consider a customer support system built on several distinct model calls chained together: one call classifies the incoming ticket's category and urgency, one call drafts a suggested reply for the human agent to review, and one call generates three alternative subject-line variants for an internal knowledge-base article summarizing the resolution. These three calls have genuinely different sampling needs, and treating them identically would be a mistake in either direction. The classification call should run at low temperature — the same ticket, routed through the same pipeline, should reliably land in the same category and urgency bucket every time, and any run-to-run drift here is pure liability, since it would mean tickets are being inconsistently triaged for reasons that have nothing to do with their actual content. The reply-drafting call plausibly wants low-to-moderate temperature as well — enough to avoid a robotic, repetitive phrasing across many tickets, but not so much that the suggested reply becomes unpredictable in tone or occasionally drifts into a register the agent has to substantially rewrite. The subject-line-variant call is the one place higher temperature earns its keep, precisely because the task's entire purpose is to produce several distinct, individually reasonable options rather than the single most probable phrasing three times over.

This is the discipline worth taking away: sampling parameters are set per call site, matched to what that specific call actually needs, not chosen once for an entire system and left untouched everywhere. A single global default — even a sensible one — is very often the wrong setting for at least one call in a multi-step pipeline.

**Why it matters for the exam:** Expect a direct conceptual question distinguishing temperature (reshaping the probability distribution's sharpness), top_p (restricting to a cumulative-probability slice that adapts to the distribution's shape), and top_k (restricting to a fixed number of top candidates regardless of distribution shape). Also expect scenario questions asking whether a described task calls for low or high temperature, based on whether output consistency or output variety is what the task actually needs.

<!-- prevnext:start -->

---

| [&larr; Previous: Prefilling and Steering Model Output](17-prefilling-and-steering-model-output) | [Next: Building a Prompt Evaluation Workflow &rarr;](19-building-a-prompt-evaluation-workflow) |
|:---|---:|

<!-- prevnext:end -->
