---
title: Module Outro
parent: Fundamentals
nav_order: 9
duration: "2:03"
---

# Module Outro
{: .no_toc }

*Read duration: 2:03*

You now have the foundation the rest of this course is built on. You've seen how large language models actually work under the hood: transformers and self-attention, tokens and context windows, the split between training and inference, and — most importantly — the fact that Claude is a probabilistic next-token predictor, which explains why prompting works, why outputs vary, why hallucination happens, and why reliability has to be engineered rather than assumed. You've drawn a precise line between the Claude API — the raw Messages API you build custom applications on — and Claude Code, the purpose-built agentic coding tool constructed on top of it, and you've practiced the judgment call of which one a given scenario calls for. And you've defined "agent" and "tool" precisely: an agent is an LLM in a loop, deciding its own next actions and observing the results, and a tool is a structured action the model can request but never directly execute itself.

That vocabulary — tokens, context windows, next-token prediction, Messages API, tool use, the orchestration loop — isn't background trivia. It's the shared language the rest of the CCA-F curriculum assumes you already speak fluently, and you'll see every one of these terms reused, tested, and built upon from here forward.

Next up is the module on the CCA-F certification itself, where you'll get oriented on the five layers the exam is actually organized around — Agentic, Tool, Workflow, Prompt, and Context — how they map to the exam blueprint, and how the rest of this course is structured to build you up through each one in turn. With the mechanics of LLMs, the API-versus-Claude-Code distinction, and the agent/tool loop now solid, you're ready to see how those pieces get organized into the five-layer architecture the certification is built around.

<!-- prevnext:start -->

---

| [&larr; Previous: Lab: Claude Architecture Selection Simulator](08-lab-claude-architecture-selection-simulator) | [Next: The CCA-F Certification &rarr;](../03-the-cca-f-certification/) |
|:---|---:|

<!-- prevnext:end -->
