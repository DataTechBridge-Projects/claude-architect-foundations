---
title: Lab: Claude Architecture Selection Simulator
parent: Fundamentals
nav_order: 8
duration: "1:49"
---

# Lab: Claude Architecture Selection Simulator
{: .no_toc }

*Video duration: 1:49*

This lab is a decision-simulation exercise: you're given a series of short scenario cards, each describing a real-world problem someone wants to solve with Claude, and for each one you have to select and justify an architecture from a small menu of options — direct Claude API integration, Claude Code (or an Agent-SDK-built custom agent), a single-agent design, or a multi-agent design with specialized sub-agents.

A typical scenario card might read: "A fintech startup wants an internal tool that watches for failed payment webhooks, investigates the likely cause across three separate internal services, and drafts (but does not send) a remediation ticket for a human to review." Working through it means asking the questions this module equips you to ask: Is this a one-shot generation task or does it need an iterative loop with real actions and observations — meaning it needs tool use? Is the investigation step complex enough, or spanning distinct enough domains, to justify splitting across multiple specialized agents rather than one generalist agent with many tools? Does the "draft, don't send" requirement point toward a specific permission or approval gate rather than full autonomy? Is this closer to a coding-agent shape (favoring Claude Code) or an embedded-product shape (favoring the raw API)?

There's no single graded "correct" architecture for most cards — the exercise is about producing a defensible chain of reasoning from constraints to design, the same skill scenario-based exam questions test. Work through each card before checking the suggested reasoning, and pay attention to which trade-offs you weighed correctly versus which ones you missed.

The cards are ordered to escalate in ambiguity. Early cards tend to have one option that's clearly better once you ask the right diagnostic questions — a one-shot summarization task, for instance, has no real case for a multi-agent design no matter how you slice it. Later cards are deliberately underspecified in ways that mirror how real requests actually arrive: a stakeholder describes what they want in outcome terms, not architecture terms, and part of the job is translating "I want a tool that does X" into a specific, defensible shape before you can even start comparing options.

Two habits from this lab carry forward into every later module. First, always name the constraint driving your choice explicitly — "multi-agent because the investigation spans three genuinely distinct service domains with different failure signatures" is a real answer; "multi-agent because it seems more thorough" is not, and the exam is built to penalize the second kind of reasoning even when it happens to land on a defensible architecture. Second, resist the pull toward the most sophisticated-sounding option by default. A single well-scoped agent with a couple of well-designed tools solves most of these scenario cards; reaching for multi-agent orchestration or full autonomy when a simpler design would do is exactly the kind of over-engineering the Additional Topics module later catalogs as a recurring antipattern.
