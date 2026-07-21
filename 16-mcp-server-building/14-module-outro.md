---
title: Module Outro
parent: MCP Server Building
nav_order: 14
duration: "2:00"
---

# Module Outro
{: .no_toc }

*Read duration: 2:00*

Start from where this module picked up: the Tool Layer had already told you what MCP is for — a standard way to expose tools, resources, and prompts to any compatible client — and this module made that concrete. Tools, resources, and prompts stopped being three words in one sentence and became three distinct primitives with their own real shapes: an invokable action, a URI-addressed piece of readable context, and a reusable, user-selected template, each earning its place for different reasons and each showing up, or not, based on whether a task genuinely needs the model's judgment to invoke it.

Building an actual server in Python turned the protocol from a diagram into code you could read line by line — a decorator deriving a schema from a function signature and a docstring, the same tool-design discipline from the Tool Layer showing up unchanged inside an MCP-specific implementation. Transport resolved into a clean choice: STDIO for a local subprocess tied to one client's session, StreamableHTTP for shared infrastructure carrying real production weight — auth, scaling, concurrent clients, audit. Sampling closed a gap you might not have noticed was open: a server borrowing the client's own model access rather than needing independent credentials. Notifications, roots, and progress reporting rounded out the protocol's supporting mechanics, keeping a long-lived, stateful connection legible and appropriately scoped.

The module closed by turning the lens around: Claude's own built-in tools need none of this — no server, no connection, no discovery — and the decision between reaching for what already ships and building or connecting something custom comes down to the same build-versus-buy logic that governs any infrastructure decision, weighed against a specific, articulable gap rather than habit in either direction.

The next module, the Workflow Layer, picks up a related thread: given a well-built set of tools — whether built-in, custom, or MCP-connected — how much of the *sequence* in which they get used should be fixed in advance through conventions, Skills, and review gates, and how much should be left to the model's own judgment call at each step.

<!-- prevnext:start -->

---

| [&larr; Previous: Choosing Built-in vs Custom Tools](13-choosing-built-in-vs-custom-tools) | [Next: Key Distinctions - MCP Server Building &rarr;](15-key-distinctions-mcp-server-building) |
|:---|---:|

<!-- prevnext:end -->
