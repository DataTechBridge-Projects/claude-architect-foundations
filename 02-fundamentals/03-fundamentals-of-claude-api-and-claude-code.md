---
title: Fundamentals of Claude API & Claude Code
parent: Fundamentals
nav_order: 3
duration: "17:10"
---

# Fundamentals of Claude API & Claude Code
{: .no_toc }

*Read duration: 17:10*

Anthropic ships Claude in several different products and interfaces, but for a CCA-F architect two of them matter above all others: the **Claude API** and **Claude Code**. They are related — Claude Code is built on top of the Claude API — but they solve different problems, and confusing them (or not understanding what one gives you that the other doesn't) is a common source of bad architecture. This lecture builds a precise mental model of each.

## The Claude API

The **Claude API** is Anthropic's programmatic interface for sending requests to Claude models and getting responses back. It is the raw, general-purpose substrate: a set of HTTP endpoints (with official SDKs in Python, TypeScript, and other languages) that any application, service, or script can call to get a Claude-generated response. If you're building a customer support chatbot, a document-summarization pipeline, a coding assistant embedded in your own IDE plugin, or a backend service that classifies support tickets, you are almost certainly building on the Claude API.

### The Messages API

The core of the Claude API is the **Messages API**. A Messages API call takes a structured list of messages — each with a role (`user` or `assistant`) and content — plus configuration like which model to use, a maximum number of output tokens, and optionally a `system` prompt that sets persistent instructions and persona for the whole conversation. Claude returns a response message containing the assistant's turn. Multi-turn conversations are handled statelessly from the API's perspective: your application is responsible for keeping track of the conversation history and re-sending the full list of prior messages (or a suitably summarized/compacted version of it) with every new call. **There is no server-side session state that persists automatically between requests** — this is a direct consequence of the training/inference distinction from the previous lecture: the model doesn't remember anything between calls unless you put it back into the context.

A basic Messages API call looks conceptually like this:

```
POST /v1/messages
{
  "model": "claude-opus-4-x",
  "max_tokens": 1024,
  "system": "You are a helpful support assistant for Acme Corp.",
  "messages": [
    {"role": "user", "content": "How do I reset my password?"}
  ]
}
```

Claude responds with an assistant message, which your application can display, log, store, and append to the message list before the next call. On top of this simple request/response shape, the Messages API supports a number of capabilities that matter for architecture: **streaming** (receiving the response incrementally as it's generated, rather than waiting for the whole thing, which matters for perceived latency in interactive applications), **tool use** (structured function-calling, covered in depth in the next lecture), **vision** (passing images as part of message content), **extended thinking** (letting the model reason step-by-step before producing a final answer on harder problems), and **prompt caching** (marking portions of a long, reused prompt — like a large system prompt or a big reference document — so the model provider can cache and reuse the processed representation across calls, cutting cost and latency for repeated context).

### Models

The Claude API gives you a choice of models, typically organized in a small family — at the time of this course, generally referred to by tiers like Haiku, Sonnet, and Opus, with version numbers appended (e.g., a specific Opus or Sonnet generation). These tiers represent different points on a capability/cost/latency trade-off: Haiku models are the fastest and cheapest, well-suited to high-volume, latency-sensitive, or simpler tasks; Opus models are the most capable, suited to the hardest reasoning and highest-stakes tasks; Sonnet models sit in between and are frequently the default choice for general-purpose agentic work because they balance capability against cost well. Choosing the right model for a given workload — rather than defaulting to the most powerful (and most expensive) one everywhere — is itself a recurring architectural decision you'll be expected to reason about on the exam.

### What the API does and doesn't give you

It's worth being explicit about scope: the Claude API, by itself, gives you a way to send text (and optionally images, tool definitions, and other structured content) to a model and get generated text back. It does not, by itself, give you a file system, a way to execute code, a terminal, persistent project memory, or an agentic loop that decides what to do next. All of that is something you build on top of the API in your own application — or something you get for free by using a product that has already built it, which is exactly what Claude Code is.

{: .important }
> Calling the Claude API does not, by itself, get you an agent. It gives you text in, text out. A file system, code execution, permissions, and an orchestration loop are scaffolding you either build yourself on top of the API, or inherit for free by using Claude Code or the Agent SDK — don't assume any of that comes bundled with a raw API call.

## Claude Code

**Claude Code** is Anthropic's agentic coding tool — available as a command-line interface, an IDE extension, and other integrations — that uses the Claude API under the hood but wraps it with a purpose-built agent loop and a set of built-in tools for working with real software projects. Where the raw API gives you "send messages, get messages back," Claude Code gives you a working coding agent: point it at a codebase, describe a task in natural language, and it will read relevant files, search the codebase, write and edit code, run shell commands (including tests, linters, and build tools), and iterate based on what it observes, largely autonomously.

Concretely, Claude Code is built from three layers stacked on the Messages API:

1. **The model itself**, called via the Claude API exactly as described above — Claude Code is not a different model, it's the same Claude models accessed through the same underlying Messages API mechanism.
2. **A set of built-in tools**, exposed to the model as callable functions: reading and writing files, searching the filesystem (glob/grep-style search), running bash commands, editing files with precise diffs, and (in many configurations) fetching web content or spawning sub-agents for parallel work. These tools are what let Claude actually *act* on a real project rather than just describe what it would do.
3. **An agentic orchestration loop** that repeatedly sends the model the current conversation and context, lets it decide whether to respond in text or invoke a tool, executes any requested tool call, feeds the result back into context, and repeats — continuing until the model produces a final answer or the task is judged complete. This loop, and the general pattern behind it, is exactly what the next lecture on agents and tools formalizes.

Claude Code also adds project-level conveniences that a raw API integration wouldn't have out of the box: a `CLAUDE.md` file mechanism for persistent project instructions and conventions that get automatically loaded into context, permission and approval settings that gate which tools can run automatically versus require human confirmation, support for custom tools and integrations via the Model Context Protocol (MCP), and slash commands and configuration for repeatable workflows. None of this is magic beyond the API — it's engineering built on top of it — but it means you get a production-grade coding agent without building the orchestration loop, tool definitions, and safety rails yourself.

### The middle ground: the Agent SDK

Between "call the raw Messages API and build everything yourself" and "use Claude Code as-is" sits a third option worth naming here, even though it's covered in more depth later in the course: the **Agent SDK**. This is Anthropic's toolkit for building your own custom agentic applications — it gives you the same kind of orchestration loop, permission model, and tool-execution scaffolding that powers Claude Code internally, but as a library you embed in your own application rather than a standalone CLI aimed at coding tasks. You'd reach for it when you want an agent with Claude Code's operational shape — an LLM in a loop, taking real actions, observing results — but for a domain other than software engineering, or when you need tighter integration with your own product's UI, data, and infrastructure than a general-purpose CLI tool can offer. It's worth keeping in your mental model now, because "raw API," "Agent SDK," and "Claude Code" form a spectrum of increasing scaffolding and decreasing flexibility, and correctly placing a described system on that spectrum is a recurring skill this course builds.

| | Claude API | Agent SDK | Claude Code |
|---|---|---|---|
| What it is | Raw Messages API | Library for building custom agents | Purpose-built agentic coding tool |
| Scaffolding provided | None — you build the loop, tools, and state yourself | Orchestration loop, permission model, tool-execution scaffolding | Full agent loop, built-in tools, CLAUDE.md, permissions, out of the box |
| Flexibility | Highest — full control over request shape and infrastructure | High — your domain, your UI, your product | Lowest — opinionated for software engineering tasks |
| Best for | Embedding Claude's language capability in your own product or pipeline | Custom agents outside coding, or tighter product integration | Working over a real codebase: reading, editing, running, iterating |

## How they relate

The relationship is layered, not parallel. Claude Code is an application built on the Claude API, in exactly the same sense that a customer-support chatbot you build yourself would be an application built on the Claude API. The API is the general-purpose foundation; Claude Code is one particular, highly polished, purpose-built product built on that foundation, specialized for software engineering tasks. This means:

- Anything Claude Code can do, you could in principle replicate by building your own agent loop against the raw Messages API — implementing your own file-read/file-write/bash tools, your own orchestration loop, your own context management. Claude Code exists so that, for the specific and extremely common use case of coding agents, you don't have to.
- Conversely, the raw API can do things that are out of scope for what Claude Code is designed for — powering a mobile app's chat feature, running as a backend microservice classifying inbound emails, or embedding Claude inside a product with no relationship to software development at all. Claude Code is not the right tool for those; you'd reach for the API directly, or for the Agent SDK (a toolkit for building your own custom agentic applications with API-level flexibility but pre-built agent-loop scaffolding).
- Both ultimately depend on the same core capability set discussed in the previous lecture: tokens, context windows, and probabilistic next-token generation. **Claude Code doesn't change any of the underlying model behavior — it changes what's *around* the model**: what tools it has access to, what gets fed into its context automatically, and how the request/response loop is orchestrated.

As an architect, the decision of "should this system be built directly against the Claude API, or should it lean on Claude Code (or the Agent SDK) as a scaffold" is one of the most common judgment calls you'll be asked to make, both in real practice and on the exam. The short version, which the next lecture expands on: reach for Claude Code (or a similar packaged agent product) when your problem looks like "an agent operating over a file system and shell, doing iterative engineering-style work," and reach for the raw API when you're embedding Claude's language capability into a custom product or pipeline where you need full control over the surrounding architecture.

## Why it matters for the exam

Expect questions that hinge on correctly identifying which layer a described scenario belongs to — for instance, recognizing that a request to "let an internal tool automatically read log files and open pull requests to fix bugs" is describing agentic, tool-using behavior that maps naturally onto Claude Code or an Agent-SDK-built equivalent, whereas "add an AI-generated product description field to our e-commerce backend" is a stateless generation task best served by a direct Messages API call. You should also be comfortable naming the concrete building blocks — Messages API, system prompts, streaming, tool use, model tiers, prompt caching — since the exam will test recognition of these terms in context, not just abstract concepts.

<!-- prevnext:start -->

---

| [&larr; Previous: Fundamentals of LLMs](02-fundamentals-of-llms) | [Next: Claude Code vs... The API? &rarr;](04-claude-code-vs-the-api) |
|:---|---:|

<!-- prevnext:end -->
