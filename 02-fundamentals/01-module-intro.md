---
title: Module Intro
parent: Fundamentals
nav_order: 1
duration: "2:31"
---

# Module Intro
{: .no_toc }

*Video duration: 2:31*

Before you can reason about agent architectures, tool design, or context management at an expert level, you need a solid floor under you: what an LLM actually is, what Claude actually is, and what "agent" actually means when Anthropic uses the word. That's what this Fundamentals module builds.

The CCA-F exam is structured around five layers — Agentic, Tool, Workflow, Prompt, and Context — and every one of them assumes you already understand the material in this module. You cannot reason correctly about "when should an agent decide to use a tool versus follow a fixed workflow" if you don't have a working mental model of how a language model actually generates output. You cannot design a good tool schema if you don't understand that, under the hood, tool use is just the model emitting structured text that your code intercepts and executes. And you cannot make an informed build decision — API versus Claude Code versus a managed agent — if you don't know what each of those actually is and where the boundaries between them sit.

This module covers three things in sequence. First, the mechanics of large language models: transformers, tokens, context windows, and the crucial fact that models are probabilistic next-token predictors rather than lookup tables or reasoning engines in the classical sense. Second, the practical distinction between the Claude API — the raw Messages API you call programmatically — and Claude Code, the agentic CLI built on top of that API with file access, bash execution, and a tool-use loop already wired up. Third, the general architecture of agents and tools: what makes something an "agent" rather than a script, and how the model, the tools, and the orchestration loop fit together.

None of this is exam trivia to memorize in isolation. It's the vocabulary and mental model that every later module — and every scenario question on the real exam — builds on. Treat this module as the foundation it is, and the rest of the CCA-F curriculum will click into place far more easily.
