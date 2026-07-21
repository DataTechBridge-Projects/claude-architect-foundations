---
title: The Docker Containers of Agentic Workflow Steps
parent: The Prompt Layer
nav_order: 11
duration: "0:26"
---

# The Docker Containers of Agentic Workflow Steps
{: .no_toc }

*Video duration: 0:26*

A Docker container is useful precisely because it doesn't leak: it declares its inputs, does its work in isolation, and produces a defined output regardless of what's happening on the host or in the container next to it. A well-designed prompt for one workflow step should behave the same way — **it shouldn't assume hidden state from an earlier step that was never explicitly passed in**, and it shouldn't silently depend on conversational residue to fill its own gaps. Treat each prompt as a container with a defined interface, and your workflow steps become swappable, testable, and debuggable in isolation, the same way containerized services are.

## A Concrete Example

Consider a three-step research pipeline: step one searches and gathers source documents, step two summarizes each source, step three synthesizes the summaries into a final report. If step two's prompt is written assuming it can see the full, meandering conversation history from step one — including offhand notes step one made about which sources looked promising, phrased informally and never formalized — then step two isn't really a separate container. It's a continuation of step one wearing a different prompt, and the moment someone edits step one's prompt (even to fix an unrelated bug), step two's behavior can shift in ways nobody predicted, because it was quietly depending on conversational residue that was never part of any documented interface. Debugging that kind of failure is miserable precisely because the two steps look independent in the codebase while being entangled in practice.

## Declared Interfaces Make Steps Swappable

The fix mirrors the fix for structure leaking into prompts, discussed elsewhere in this module: make the interface explicit rather than implicit. Step two's prompt should declare exactly what it expects to receive — a list of source documents with a defined shape, nothing more — and should be written and testable as if step one had never existed as a conversation, only as a data producer. Do that consistently across a pipeline, and each step becomes independently swappable: you can replace the summarization step with a different model, a different prompt, or a cheaper approach entirely, and as long as it honors the same declared input and output shape, nothing downstream needs to change or even know. It also becomes independently testable — you can feed step two a synthetic input without ever running step one — and independently debuggable, because a failure can be localized to the one container that produced bad output rather than requiring you to replay an entire multi-step conversation to find where things went wrong.

**Why it matters for the exam:** Expect questions describing a multi-step agentic workflow where changing one step's prompt unexpectedly breaks a downstream step, and asking what design principle was violated. The answer is a missing explicit interface between steps — hidden dependence on conversational context or unstated assumptions — rather than anything wrong with either prompt in isolation.

<!-- prevnext:start -->

---

| [&larr; Previous: Batch Processing](10-batch-processing) | [Next: Independent & Multi-Pass Review &rarr;](12-independent-and-multi-pass-review) |
|:---|---:|

<!-- prevnext:end -->
