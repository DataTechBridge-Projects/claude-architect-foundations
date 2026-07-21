---
title: Module Intro
parent: The Tool Layer
nav_order: 1
duration: "3:00"
---

# Module Intro
{: .no_toc }

*Video duration: 3:00*

The Agentic Layer gave you the loop: model call, tool-use request, execution, result, repeat. That loop is the skeleton. The Tool Layer is about the thing the model actually reaches for at step three of every iteration — the tool itself — and it asks a deceptively simple question: what makes a tool *good*? Not good in the sense of "well-implemented code," but good in the sense that a model, reading nothing but a name, a description, and a schema, reliably picks the right one, fills it in correctly, and knows what to do when it fails.

That reframing is the spine of this module. You've designed APIs before, or at least used them, and you already have instincts about what makes an endpoint well-designed — clear naming, sensible parameters, useful error messages. Almost all of that instinct transfers here, with one crucial twist: the caller reading your documentation isn't a human engineer who can tolerate ambiguity by asking a colleague or checking a Slack channel. The caller is a language model, mid-inference, choosing among however many tools you've exposed to it, using only the text you gave it to decide. Every word in a tool's name and description is doing real architectural work, not just documentation work.

This module walks that idea through the full lifecycle of a tool call. It starts with **definition and selection** — how naming, descriptions, and schemas determine whether a model picks correctly among several similar-looking tools, and the perennial tension between too many narrow tools and too few overloaded ones. It covers the choice between rigid structure and free-text prose within a schema, and when each is the right call. It spends real time on **error response design**, because a tool that fails is far more common in practice than a tool that always succeeds, and how that failure is communicated determines whether the model recovers gracefully or spirals. From there it moves to **error recovery** at the orchestration level — retries, fallbacks, escalation — and to **MCP**, the Model Context Protocol, as the standard mechanism by which tools get exposed to Claude at all, along with the governance questions a growing tool ecosystem forces on any organization. It closes by grounding all of this in a concrete, already-shipped example: Claude Code's own built-in tools, examined as a worked case study in the principles you'll have just learned.

By the end, you should be able to look at a tool definition — or a tool failure — and diagnose exactly what's wrong with it using this module's vocabulary, the same way the Agentic Layer taught you to diagnose a malfunctioning loop.

<!-- prevnext:start -->

---

| [&larr; Previous: The Tool Layer](./) | [Next: Tool Definition & Selection &rarr;](02-tool-definition-and-selection) |
|:---|---:|

<!-- prevnext:end -->
