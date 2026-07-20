---
title: Easier for "Machine Talk"
parent: The Prompt Layer
nav_order: 7
duration: "0:23"
---

# Easier for "Machine Talk"
{: .no_toc }

*Video duration: 0:23*

When a model's output is read by a human, readability is what matters — natural paragraphs, friendly headers, a little redundancy for clarity. When output feeds straight into another system — a parser, a database write, the next agent in a pipeline — readability is a cost, not a benefit. Terse, rigidly structured, unambiguous formats parse faster, fail less often, and carry no wasted tokens on pleasantries no downstream code will ever read. Once you know the consumer is a machine, optimize deliberately for "machine talk," and stop spending effort making output pleasant for a human who will never see it.

## A Concrete Example

Consider a step in a pipeline that extracts a customer's shipping address from a support ticket and passes it to a downstream fulfillment system. A human-oriented response might read: "I found the shipping address in the ticket. It looks like the customer's address is 42 Baker Street, Apartment 3, Springfield, and I believe the ZIP code is 62704, though it's a little hard to tell from the formatting." That response is pleasant to read and entirely useless to a parser, which now has to somehow extract five fields out of a sentence hedged with "I believe" and "a little hard to tell." The machine-talk version is simply: `42 Baker Street | Apt 3 | Springfield | IL | 62704` — or, better still, the equivalent structured JSON object. No hedging, no narration about the extraction process, nothing for a parser to trip over.

## The Cost Compounds at Scale

This isn't just about avoiding parse failures — it's also about cost and latency, which matter more the more a pipeline runs. Every sentence of conversational padding is tokens generated that a machine consumer discards unread, and at the scale of thousands or millions of pipeline invocations, that padding adds up to real, avoidable spend and real, avoidable latency on the critical path. This is the same discipline the lecture on achieving structured outputs pushes toward with schema-constrained generation and the tool-call-as-output pattern: once you know for certain that the consumer of a given output is code, not a person, every token spent on making that output "friendly" is a token spent on nothing. This becomes especially important in multi-step agentic workflows, where the output of one step is the input to the next — treating each step's output as machine talk by default, and reserving natural-language framing only for the final step that actually surfaces to a human, keeps the whole pipeline both cheaper and more reliable.

**Why it matters for the exam:** Expect a question contrasting two candidate output formats for the same underlying data, one narrative and one terse or structured, and asking which is appropriate for a given consumer. The signal to look for is who or what reads the output next — a human user gets natural language, a parser or another agent gets the leanest structured format that carries the same information.
