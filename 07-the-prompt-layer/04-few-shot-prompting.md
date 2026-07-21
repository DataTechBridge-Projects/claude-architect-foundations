---
title: Few-Shot Prompting
parent: The Prompt Layer
nav_order: 4
duration: "16:17"
---

# Few-Shot Prompting
{: .no_toc }

*Video duration: 16:17*

Instructions tell a model what to do in words. Examples show it. **Few-shot prompting** is the technique of including a small number of worked examples — input paired with the desired output — directly in the prompt, so the model can infer the pattern you want by demonstration rather than (or in addition to) by description. It's one of the oldest and most reliable prompting techniques precisely because it sidesteps a hard problem: some things are genuinely difficult to specify precisely in prose, but trivially easy to show.

## When Examples Beat Instructions

Not every task benefits equally from few-shot examples, and knowing when to reach for them is itself an architectural judgment call. Examples earn their place in a prompt when the task has properties that are hard to fully articulate in words but easy to demonstrate: a specific tone or voice, a formatting convention with idiosyncratic details, a classification boundary that's fuzzy in the abstract but clear in specific cases, or a domain-specific pattern that would take paragraphs of prose to describe precisely and that a single well-chosen example conveys in a few lines.

Take tone as a concrete case. You can write pages of instructions trying to describe "friendly but professional, concise, never apologetic unless we're actually at fault" — and a model will make a reasonable attempt at honoring all of that. But show it three real examples of past responses that hit exactly that register, and it will match the pattern far more precisely than any description could achieve, because tone is a "you know it when you see it" property, and examples let the model see it directly instead of reconstructing it from an approximation in words.

The same logic applies to structured extraction with idiosyncratic conventions — say, extracting line items from invoices where quantities are sometimes embedded in a product description string rather than a separate field, and you have a house convention for how to normalize that. Describing every such edge case in prose is exhausting and incomplete; a handful of examples that include exactly those edge cases teaches the pattern far more efficiently. As a rule of thumb: if you find yourself writing several sentences trying to describe a stylistic or structural nuance, stop, and ask whether a well-chosen example would communicate the same nuance in less space and with less ambiguity.

Conversely, few-shot examples are the wrong tool — or at least an inefficient one — for tasks that are already unambiguous once stated plainly. "Extract the total dollar amount from this invoice and return it as a number" doesn't need three worked examples; it needs a clear instruction and maybe one example to nail down the exact output format. Piling examples onto an already-clear instruction adds token cost and prompt length without buying you meaningfully more reliability, and in the worst case it can actively narrow the model's behavior in ways covered below.

| Reach for few-shot examples when... | Reach for plain instructions when... |
|---|---|
| Tone or voice is hard to describe in prose | The task is already unambiguous once stated |
| A formatting convention has idiosyncratic details | One sentence fully specifies the output format |
| A classification boundary is fuzzy in the abstract | The boundary is already crisp and objective |
| A domain pattern would take paragraphs to describe | The instruction alone reliably produces the right output |

## Covering the Range Without Overfitting

The central design challenge in few-shot prompting isn't whether to include examples — it's which examples to include, and this is where most few-shot prompts go wrong. The instinct is to reach for two or three examples that look like the "typical" case, because those are the easiest to write and feel representative. But a model learns from the *pattern across* your examples, not just their individual content, and if all your examples share an unstated commonality — similar length, similar structure, similar level of complexity — the model will often absorb that commonality as part of the pattern, even though you never intended it as a requirement.

The fix is to choose examples that deliberately span the **range** of inputs the system will actually encounter in production, not just the center of that range. If a classification task will see both short, unambiguous inputs and long, messy, borderline ones, your few-shot set should include at least one of each — not three near-identical "clean" cases. If a summarization task will sometimes receive a two-paragraph memo and sometimes a forty-page report, showing only short-document examples will bias the model toward producing short-document-style summaries even when it's handed something much longer, because "roughly this length" was implicitly part of the pattern it inferred.

This means the process of selecting few-shot examples is really a miniature exercise in understanding your input distribution. Before you can choose good examples, you need to know: what does the full range of realistic inputs actually look like? What are the edge cases that show up rarely but matter when they do — the malformed input, the ambiguous case, the one that technically fits the category but only barely? A well-constructed few-shot set usually includes at least one edge case deliberately, precisely so the model sees that the pattern extends there too, rather than assuming edge cases are out of scope because every example it saw was a clean, central case.

A useful practical habit: after drafting your few-shot examples, explicitly list the dimensions along which real inputs vary — length, formality, domain vocabulary, completeness, ambiguity — and check your example set against that list. If a dimension varies significantly in production but your examples don't vary along it at all, that's a gap worth closing before the prompt ships, not after a production failure surfaces it.

## The Risk of Teaching the Wrong Lesson

The subtlest and most dangerous failure mode in few-shot prompting is the example that accidentally teaches a lesson you never intended, through some incidental detail that has nothing to do with the actual pattern you're trying to demonstrate. Models are excellent pattern-matchers, and they don't reliably distinguish between "this detail is the point" and "this detail just happened to be true of my example."

A classic version of this: you're building a few-shot prompt for classifying customer support tickets by urgency, and every one of your "high urgency" examples happens to mention a dollar amount, because the examples you grabbed to write the prompt quickly came from billing disputes. Nothing in your instructions says urgency is about money. But if every high-urgency example contains a dollar figure and every low-urgency example doesn't, the model may quietly learn "mentions money → high urgency" as part of the pattern, and then misclassify a genuinely urgent security report that happens not to mention a dollar amount, or over-classify a low-stakes billing question that does. The incidental correlation in your examples became, functionally, part of the specification — even though you never wrote it down and might not even notice you'd introduced it.

This risk compounds with formatting details. If every example in your set happens to be written in the same sentence length, uses a particular greeting, or is sourced from the same product line, the model may bind the target behavior to those incidental features rather than the actual signal you intended. The failure is hard to catch by inspection because each individual example looks fine in isolation — the problem only exists in the aggregate, as a correlation across the set that you didn't intend and may not even perceive, since as the author you already know the "real" reason you chose each example and aren't looking for spurious ones.

Guarding against this requires actively varying the incidental details across your examples while holding the actual signal constant. If urgency is the real target, deliberately include a high-urgency example with no dollar amount and a low-urgency example that does mention money, so the model can't take the shortcut of keying off the wrong feature. If tone is the target but your examples are drawn from different product lines, make sure both a "correct" and an "incorrect" tone example exist within the same product line, so product identity can't become an accidental proxy for the judgment you're trying to teach. In general: for every example, ask "what does this example teach the model that I didn't intend to teach it?" — and if the honest answer includes anything besides the target pattern, either fix the example or add a counter-example that breaks the spurious correlation.

## Practical Mechanics

A few structural notes round out the practice. Few-shot examples are typically most effective when formatted consistently and clearly delimited — using consistent tags, labels, or a repeated structure (`Input: ... / Output: ...`) so the model can reliably identify where one example ends and the next begins, and where the actual task to solve starts once the examples are done. Consistency of *format* across examples is good; consistency of *content characteristics* (length, tone, difficulty) is the trap discussed above. Keep the format rigid and the substance varied.

The number of examples matters less than their selection quality. Two or three well-chosen examples that span the input range and avoid spurious correlations will consistently outperform six or eight examples that are all minor variations on the same theme, and the shorter set costs less in tokens and latency on every single call. There's rarely a reason to keep adding examples once you've covered the meaningful range of variation — additional examples that don't add new information mostly add cost.

Finally, few-shot examples are not "set once and forget." As production traffic reveals new edge cases or new failure patterns, the example set should be revisited and updated — this is exactly the kind of change that belongs in a versioned prompt history, since swapping an example can shift model behavior in ways that are worth being able to trace and, if necessary, roll back.

## Few-Shot Examples as a Debugging Tool, Not Just a Starting Tool

It's worth explicitly noting that few-shot examples aren't only something you write once at the start of a project — they're one of your most useful levers when a deployed prompt starts misbehaving in a specific, identifiable way. If batch monitoring (covered later in this module) surfaces a recurring misclassification on a particular kind of input, the fastest fix is often not a paragraph of new instructions but a single, precisely chosen example that demonstrates the correct handling of exactly that input pattern. This is faster to write, easier to verify, and less likely to introduce new ambiguity than trying to describe the fix in prose — and it's a good illustration of the broader principle from this lecture: reach for an example when the thing you're trying to communicate is easier to show than to explain, and a production failure pattern is very often exactly that kind of thing.

The reverse move matters too. If a newly added example doesn't measurably improve the failure rate on the pattern it was meant to fix — or worse, if it introduces a new incidental correlation that degrades some other slice of inputs — that's a signal to remove or revise it rather than let the example set grow indefinitely. A bloated set of examples added reactively, one per incident, without ever being pruned or re-evaluated as a whole, tends to accumulate exactly the kind of unintended correlations this lecture has spent most of its time warning against — because each individual addition looked reasonable in isolation, and nobody stepped back to check whether the set as a whole still avoided teaching the wrong lesson.

**Why it matters for the exam:** Expect questions that present a few-shot prompt and ask you to identify what unintended pattern the model is likely to learn from it — usually by pointing out a correlation across the examples that has nothing to do with the stated task. Also expect questions distinguishing when few-shot examples are the right tool versus when clear instructions alone would suffice, and questions testing whether you recognize an example set that clusters around the "typical" case while ignoring the edge cases production will actually encounter.

<!-- prevnext:start -->

---

| [&larr; Previous: Structure Is Leaking Into Prompts!](03-structure-is-leaking-into-prompts) | [Next: Ruthlessly Vary... Within the Expected Range &rarr;](05-ruthlessly-vary-within-the-expected-range) |
|:---|---:|

<!-- prevnext:end -->
