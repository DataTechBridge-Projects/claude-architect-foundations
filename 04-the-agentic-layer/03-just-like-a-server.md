---
title: Just Like a Server
parent: The Agentic Layer
nav_order: 3
duration: "0:24"
---

# Just Like a Server
{: .no_toc }

*Video duration: 0:24*

A useful mental shortcut for the orchestration loop: think of each model call as a **server request-response cycle**. The request is the assembled context (state passed in); the response is either an answer or a tool-use request (an instruction, not unlike an RPC call, for the surrounding system to execute). The model itself holds no memory between calls — like a stateless server handler, everything it "knows" for that call arrives in the request. State lives outside the model, in whatever transcript or store re-sends context on the next call. This framing will recur throughout the module.

Push the analogy one level further and it maps cleanly onto real HTTP semantics. The system prompt, conversation history, and tool definitions play the role of the request body; a plain-text answer is like a `200 OK` carrying a finished payload; a tool-use request is closer to a `202 Accepted` — the server (the model) hasn't produced a final result yet, it's handed back a structured instruction that something else has to execute before the exchange can continue. Just as an HTTP server never reaches out and touches a database directly on the client's behalf without being asked, the model never reaches out and touches a file system or a network on its own — it only ever emits tokens describing the action it wants taken, and the harness is the layer that actually performs it.

Statelessness cuts both ways, and it's worth naming the consequence, not just the shape. On the upside, a stateless handler is trivially retriable and horizontally scalable: any available model instance can serve the next call in a conversation because nothing about "this session" lives inside the model itself, the same way any web server behind a load balancer can serve the next HTTP request because session state lives in a database or cache, not in the process handling the request. On the downside, it means the model has zero implicit memory to fall back on — if a fact, a constraint, or an earlier decision isn't explicitly re-sent in the next request's context, it is, for all practical purposes, gone. There's no partial recall, no "I think I remember something about this" the way a stateful system might retain a cache; the model reasons only from what's in front of it.

This is also why the safeguarding patterns covered later in this module (checkpoints, idempotency, rollback — see "Back to the Server Analogy...") have to live in the surrounding system rather than in the model: a stateless handler can't be the thing responsible for guaranteeing exactly-once effects, because it has no durable memory of what it already did. And it's why the timelines lecture later in this module matters — a single request-response cycle is the shortest timeline in the system, nested inside longer-lived constructs (subagents, tasks, sessions) that actually persist the state a stateless model call can't.

<!-- prevnext:start -->

---

| [&larr; Previous: The Orchestration Loop](02-the-orchestration-loop) | [Next: Structured Tool Use &rarr;](04-structured-tool-use) |
|:---|---:|

<!-- prevnext:end -->
