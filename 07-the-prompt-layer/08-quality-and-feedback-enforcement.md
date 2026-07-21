---
title: Quality and Feedback Enforcement
parent: The Prompt Layer
nav_order: 8
duration: "19:20"
---

# Quality and Feedback Enforcement
{: .no_toc }

*Video duration: 19:20*

A model that produces output without ever checking that output against a standard is operating open-loop: it generates once, and whatever comes out is what you get. **Quality and feedback enforcement** is the set of techniques for closing that loop — building self-checking into the prompt or the surrounding process so that low-quality output has a chance of being caught and corrected before it ever reaches a downstream system or a human who trusted it. This is one of the most consequential ideas in the entire Prompt Layer, because it's the difference between a system that's **reliable by accident and one that's reliable by design**.

## Why Self-Checking Works

It might seem strange that a model checking its own output would catch anything at all — if it were capable of getting the task right, wouldn't it have gotten it right the first time? The answer is that generation and evaluation are different cognitive tasks, even for the same model, and a model is often measurably better at judging a completed piece of work against explicit criteria than it is at spontaneously producing that same quality on a single unguided pass.

There are a few reasons this asymmetry holds up in practice. First, generation happens under time pressure in a real sense — the model is producing tokens sequentially without the ability to revise earlier tokens once written, so early choices (an opening sentence, a structural decision) can lock in a suboptimal path that only becomes visible once the output is complete and viewable as a whole. Evaluation, by contrast, looks at the finished artifact all at once, the way an editor reviews a finished draft rather than watching over a writer's shoulder sentence by sentence. Second, an evaluation pass can be given a narrower, more specific job than the original generation pass — "does this output violate any of these five rules?" is a much more constrained question than "write a good response," and models tend to be more reliable on narrow, checkable questions than on open-ended production. Third, explicitly separating "produce" from "check" gives you a natural point to inject criteria the model might not have been holding actively in mind while generating — a rubric it can compare the finished output against, rather than trying to satisfy from memory while also handling everything else the generation task demands.

## Rubrics and Constitutions

The most direct implementation of self-checking is to give the model an explicit **rubric** — a list of specific, checkable criteria the output should satisfy — and ask it to evaluate its own draft against that rubric before finalizing a response, or as a genuinely separate step after a first draft exists. A rubric for a customer-facing email draft might include: does it address the customer's actual question, does it avoid promising a specific resolution timeline, does it stay under 150 words, does it avoid internal jargon, is the tone consistent with brand voice. Each item is phrased so it can be answered with something close to yes/no, which makes the self-check tractable rather than another vague, open-ended judgment call.

A related but broader pattern is the **constitution**: a standing set of principles the model should hold itself to across an entire class of tasks, not just one specific output. Where a rubric is usually task-specific ("does this particular email satisfy these particular criteria"), a constitution is more like a durable set of values or non-negotiable constraints — "never recommend an action that would violate the customer's stated budget," "never present a low-confidence extraction as if it were verified," "always flag when required source data was missing." Constitutions tend to live at the system-prompt level, since they apply across every invocation, while task-specific rubrics are often injected at the point where a particular output needs checking.

The practical value of both patterns is the same: they convert "produce good output" — an instruction the model has to interpret and hold in mind unaided — into "check this specific list of concrete things," which is a task the model can execute much more mechanically and therefore much more reliably. Vague quality bars produce vague adherence; specific, enumerated criteria produce something much closer to consistent enforcement.

## Catching Low-Quality Output Before It Propagates

The architectural payoff of self-checking is upstream containment: catching a problem at the point of generation is dramatically cheaper than catching it after it has already flowed into the next step of a pipeline, been acted on by another agent, or reached a customer. This is the same principle that makes early validation valuable in traditional software — a bug caught by a unit test costs a few minutes; the same bug caught in production after it has corrupted downstream data can cost days of cleanup and, in some domains, real reputational or financial damage.

In an agentic system, this containment matters even more than in a typical software pipeline, because agents often act on the outputs of previous steps with a degree of trust that a human reviewer wouldn't extend automatically. If Step 1 in a pipeline silently produces a slightly wrong classification, and Step 2 treats that classification as ground truth without questioning it, the error doesn't just persist — it can actively compound, because Step 2's own output is now built on a false premise, and Step 3 inherits both errors. A self-check inserted right after Step 1, checking the classification against a rubric before it's allowed to flow forward, is a cheap circuit breaker against exactly this kind of compounding failure, and it's far cheaper than trying to detect and untangle the compounded error several steps later.

This is also where quality enforcement connects directly to escalation: a self-check isn't just a pass/fail gate that either lets output through or silently discards it. A well-designed self-check that fails should route to a defined fallback — retry with the identified problem called out explicitly, escalate to a human reviewer, or return a clear "unable to complete confidently" signal rather than forcing output through regardless. The goal isn't just detecting low quality; it's making sure **detection actually changes what happens next**, rather than being logged and ignored.

{: .important }
> A self-check that fails and has nowhere to route is nearly worthless. Never build a quality gate without also defining what happens on failure — retry, escalate, or a clear "not confident" signal — or the check becomes something that gets logged and ignored rather than something that actually changes the outcome.

## Techniques for Structuring the Self-Check

Several concrete patterns are common in practice, and they aren't mutually exclusive — production systems often layer more than one.

**Inline self-critique before finalizing.** The prompt instructs the model to draft a response, then explicitly review that draft against stated criteria, then produce a final version — all within a single generation, using the model's own reasoning process as the checking mechanism. This is cheap (no extra API call) but weaker than a separate pass, because the same context and the same momentum that produced the flawed first draft is still influencing the "independent" review of it.

**Structured rubric scoring.** Rather than an open-ended critique, the model is asked to score the output against each rubric item explicitly — yes/no, or a numeric scale — producing a structured evaluation that your application code can inspect programmatically and act on (route to a human if any critical item scores no, retry automatically if a specific fixable item failed, pass through cleanly if everything scores well). This connects directly back to the structured-output techniques from the previous lecture: an evaluation is far more useful to your system when it's a parseable verdict rather than a paragraph of prose you'd have to re-read to determine the outcome.

**Confidence and uncertainty signaling.** Rather than a binary pass/fail, the model is asked to flag its own confidence in specific claims or decisions within the output — which fields it's certain about versus which were inferred or guessed. This is especially valuable for extraction and classification tasks, where "I'm not confident about this one field" is actionable information a downstream system can use to route that specific case for human review, rather than treating the entire output as uniformly trustworthy or uniformly suspect.

**Separate reviewer pass.** The strongest version of self-checking isn't really "self" at all — it's a genuinely separate prompt, sometimes even a separate model call with a different framing or persona, dedicated entirely to evaluating output that a prior pass produced. This is powerful enough, and different enough in character from the inline techniques above, that it gets its own full treatment later in this module under independent and multi-pass review; the short version here is that a fully separate pass avoids the "grading its own homework" problem that inline self-critique doesn't fully escape.

| Technique | Independence | Relative Cost |
|---|---|---|
| Inline self-critique | Weakest — same context and momentum as the draft | Lowest — no extra API call |
| Structured rubric scoring | Moderate — explicit, checkable criteria | Low to moderate |
| Confidence/uncertainty signaling | Moderate — flags what needs review | Low to moderate |
| Separate reviewer pass | Strongest — genuinely independent | Highest — a full extra model call |

## The Cost and Latency Tradeoff

None of this is free, and pretending otherwise is itself a design mistake. Every additional check — whether it's inline self-critique, structured rubric scoring, or a fully separate reviewer pass — adds tokens, adds latency, and in the case of a separate pass, adds an entire additional model call with its own cost. An architect has to weigh that cost against what a missed quality failure would actually cost if it reached production unfiltered.

The right calibration depends heavily on stakes. A low-stakes, high-volume task — generating short internal summaries that a human skims and can easily correct — often doesn't justify heavyweight verification; the cost of an occasional miss is low, and the cost of verifying every single one at scale adds up fast for little marginal benefit. A high-stakes, lower-volume task — drafting a customer-facing legal notice, or extracting figures that feed directly into a financial report — usually justifies real verification investment, because the cost of one bad output reaching its destination unfiltered can dwarf the cumulative cost of checking every single one.

A useful way to reason about this trade-off explicitly: estimate the cost of a verification pass (in tokens, latency, and engineering complexity) against the expected cost of an undetected failure (probability of failure times the cost of that failure reaching production), and choose the level of verification — none, inline self-critique, structured rubric, or full separate-pass review — that makes sense for that specific ratio, rather than defaulting to either extreme out of habit. It's just as much a design mistake to bolt heavyweight multi-pass review onto a low-stakes task as it is to ship a high-stakes task with no verification at all — both represent a mismatch between the level of rigor applied and the level actually warranted.

## A Worked Example

Consider a system that drafts customer-facing responses to billing disputes, where the draft is sent to the customer automatically without a human in the loop for routine cases. This is exactly the kind of high-stakes, unattended task that justifies real verification investment, and it's worth walking through what a layered self-check looks like concretely.

The generation prompt produces a draft response. Before that draft is allowed to send, it's checked against a structured rubric with items like: does the response commit to a specific dollar amount or refund without confirming it against the actual account balance; does it promise a resolution timeline the team can't actually guarantee; does it use a tone consistent with brand voice; does it avoid disclosing another customer's information; is it free of any internal-only terminology or system names that shouldn't reach an external audience. Each item is scored yes/no by a structured evaluation pass, not left as an open-ended "does this look okay?" impression, precisely because a structured yes/no per item is something application code can act on directly — routing to a human reviewer the moment any critical item scores no, rather than requiring a human to re-read every single rubric result themselves.

Two of those items are worth distinguishing by consequence. A tone violation is low-stakes and easily fixed automatically — the system can simply regenerate with tone flagged as the specific issue, and try again. A dollar-amount or timeline violation is high-stakes and not something to silently auto-correct, because auto-correcting a financial commitment without human confirmation just relocates the risk rather than removing it — that item should route straight to a human, every time, with no automatic retry. This is the cost/latency tradeoff made concrete: the system spends a small, fixed cost checking every draft against the full rubric, but reserves the more expensive step — a human's time — for exactly the subset of cases where the rubric flags something that actually warrants it, rather than either checking nothing (and risking a bad commitment reaching a customer) or routing everything to a human (and eliminating the automation's value entirely).

**Why it matters for the exam:** Expect scenario questions that describe a pipeline producing occasional bad output and ask what level of verification is appropriate given the stated stakes — the correct answer usually isn't "add maximum verification" but "add verification proportional to what a failure would cost." Also expect questions distinguishing inline self-critique from a genuinely independent reviewer pass, and why the latter more reliably avoids the failure mode of a pass validating its own flawed assumptions.

<!-- prevnext:start -->

---

| [&larr; Previous: Easier for "Machine Talk"](07-easier-for-machine-talk) | [Next: "Traditional" Sanitization &rarr;](09-traditional-sanitization) |
|:---|---:|

<!-- prevnext:end -->
