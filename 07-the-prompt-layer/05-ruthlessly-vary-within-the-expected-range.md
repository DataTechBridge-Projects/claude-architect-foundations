---
title: Ruthlessly Vary... Within the Expected Range
parent: The Prompt Layer
nav_order: 5
duration: "0:30"
---

# Ruthlessly Vary... Within the Expected Range
{: .no_toc }

*Video duration: 0:30*

A tempting shortcut when writing few-shot examples is to grab two or three that were easy to find and call it done. Resist it. Examples that cluster near-identical — similar length, similar phrasing, similar difficulty — **teach the model that narrow cluster is the whole pattern**, not the actual range of inputs production will send it. Ruthlessly vary your examples across every dimension that matters downstream — length, formality, ambiguity, edge-case status — while staying strictly within the range you actually expect. Variety inside the expected range builds robustness; variety outside it just confuses the model about the task.

## A Concrete Example

Suppose you're building a few-shot prompt to classify support tickets by urgency, and the three examples you grab happen to be: a two-paragraph ticket about a payment failure, another two-paragraph ticket about a payment failure with a different dollar amount, and a third about a login issue that's also roughly two paragraphs. On paper you have three examples covering two categories. In practice, every example is medium-length, formally written, and two of the three link urgency to money. Feed that prompt a one-line ticket ("app is down, losing customers, help") or a rambling three-paragraph ticket where the deadline is buried in the second sentence, and accuracy degrades — not because the model can't handle those inputs, but because nothing in the examples showed it what urgency looks like outside the narrow shape it was shown.

## Spurious Correlation Is the Real Danger

The more subtle failure mode isn't just narrow coverage — it's the wrong lesson variety was supposed to prevent. If every "urgent" example in your few-shot set happens to mention a dollar figure, the model has no reliable way to distinguish "urgent" from "mentions money," because those two signals were never pulled apart in the data you showed it. A ticket that's genuinely urgent but has no financial language at all — a security breach, a total outage — can get silently misclassified, and a ticket that mentions a large dollar amount in a routine, non-urgent context can get over-classified. Deliberately including an urgent example with no financial language, and a non-urgent example that does mention money, breaks the spurious correlation before it has a chance to form. This exact failure mode is what the module's hands-on lab is built around: a fragile prompt whose few-shot examples taught an incidental correlation instead of the actual classification rule, discoverable only by running a properly varied test set and watching where it breaks.

**Why it matters for the exam:** Expect questions describing a few-shot prompt that performs well on examples similar to its own shots and poorly on the rest of production traffic. The correct diagnosis is narrow or correlated example coverage, and the fix is deliberately varying examples across the dimensions that matter — not simply adding more examples that look like the ones you already have.

<!-- prevnext:start -->

---

| [&larr; Previous: Few-Shot Prompting](04-few-shot-prompting) | [Next: Achieving Structured Outputs &rarr;](06-achieving-structured-outputs) |
|:---|---:|

<!-- prevnext:end -->
