---
title: Module Intro
parent: MCP Server Building
nav_order: 1
duration: "2:30"
---

# Module Intro
{: .no_toc }

*Read duration: 2:30*

The Tool Layer already gave you MCP's shape from a distance: a server exposes tools, resources, and prompts; a client discovers and connects; governance keeps a growing ecosystem of servers honest. That lecture answered *what MCP is for*. This module answers *how you actually build one* — and, just as importantly, *when you'd bother*.

It starts by taking the three primitives that lecture named only in passing — tools, resources, prompts — and giving each its own real treatment: what distinguishes an invokable action from a piece of context you can simply read, and why a template-shaped prompt is a different kind of thing from either. From there the module gets concrete fast, walking through an actual minimal MCP server in Python: how you stand one up, how you register a tool against a schema, and what runs underneath a decorator that makes a plain Python function callable by a model it has never directly seen.

Two more sections dig into the primitives from the server-authoring side — schema design for tools exposed specifically through MCP, and the URI-addressed, read-only shape of a resource — before the module turns to the other half of the relationship: how a client discovers what a server offers and surfaces that to the model as ordinary tool use. Transport gets its own real treatment too, since STDIO and remote, HTTP-based transport solve genuinely different problems for genuinely different reasons, with real production consequences — auth, scaling, concurrent clients — once a server leaves a developer's own machine. Two quieter primitives round out the protocol's mechanics: sampling, where a server borrows the connecting client's model access instead of needing its own, and the supporting cast of notifications, roots, and progress reporting that keep a long-running, stateful connection honest.

The module closes by turning the lens around entirely: Claude's own built-in tools, which need no MCP server at all, and a decision framework for when reaching for MCP is the right architectural call versus over-engineering a problem a built-in tool already solves. By the end, MCP should stop being a diagram you can describe and start being a protocol you could actually implement against, on either side of the connection.

<!-- prevnext:start -->

---

| [&larr; Previous: MCP Server Building](./) | [Next: Tools, Resources & Prompts: The Three MCP Primitives &rarr;](02-tools-resources-and-prompts-the-three-mcp-primitives) |
|:---|---:|

<!-- prevnext:end -->
