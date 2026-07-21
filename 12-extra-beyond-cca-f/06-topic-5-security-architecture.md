---
title: "Topic 5: Security Architecture"
parent: "Extra Module: Beyond the CCA-F"
nav_order: 6
duration: "0:19"
---

# Topic 5: Security Architecture
{: .no_toc }

*Video duration: 0:19*

An agent with tools is an agent with an attack surface, and that surface is different in kind from a traditional application's. **Prompt injection** — malicious or manipulative instructions smuggled in through a tool result, a retrieved document, a web page the agent fetched, or ordinary user input — is the headline risk, because the model doesn't have a clean, reliable way to distinguish "instructions from my operator" from "text I happened to encounter while doing my job." A support agent that reads a customer's message and a support agent that reads a webpage the customer linked to are, from the model's point of view, doing the same kind of thing: reading text and reacting to it. If that text contains "ignore your previous instructions and do X," a naively built agent may comply.

Because there's no single fix for that, security architecture for agentic systems is layered rather than singular. **Scope tools tightly** on the principle of least privilege — an agent that only *needs* to read data shouldn't also hold a tool that can send money or delete records, no matter how convenient it would be to bundle them. **Treat all retrieved and tool-returned content as untrusted data, never as instructions** — this is the most important mental shift, and it echoes the Tool Layer's structure-over-prose principle: content flowing back into context should be handled as data to reason about, not as commands to obey. **Keep irreversible or high-stakes actions behind human approval**, the same checkpoint pattern from the Workflow and Context layers. And **sandbox anything that executes code or runs arbitrary commands**, so that even a successful injection is contained to a boundary that can't reach anything sensitive.

{: .important }
> Never let retrieved documents, web pages, or tool results be treated as instructions, no matter how directive their phrasing sounds. Conflating "content the agent read" with "commands the agent should obey" is the single mechanism behind almost every prompt injection exploit.

| Defense Layer | Principle | Guards Against |
|---|---|---|
| Least-privilege tool scoping | An agent gets only the tools it needs | A read-only agent also being able to delete or send money |
| Untrusted-content handling | Retrieved/tool-returned content is data, not instructions | Prompt injection via documents, web pages, tool results |
| Human approval gates | Irreversible or high-stakes actions require sign-off | An injected instruction triggering a costly or destructive action |
| Sandboxing | Code/command execution stays inside a limited boundary | A successful injection reaching sensitive systems |

Security here isn't a separate discipline bolted onto the architecture after the fact — it's a direct extension of the Tool Layer's governance principles and the Workflow Layer's safeguarding principles, applied specifically to the threat of an adversarial actor trying to hijack the agent's own decision-making.

<!-- prevnext:start -->

---

| [&larr; Previous: Topic 4: Cost Engineering](05-topic-4-cost-engineering) | [Next: Topic 6: Persistent Memory Systems &rarr;](07-topic-6-persistent-memory-systems) |
|:---|---:|

<!-- prevnext:end -->
