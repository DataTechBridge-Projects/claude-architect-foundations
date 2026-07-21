---
title: Key Distinctions - MCP Server Building
parent: MCP Server Building
nav_order: 15
duration: "4:00"
---

# Key Distinctions - MCP Server Building
{: .no_toc }

*Video duration: 4:00*

A condensed pass through the module's highest-yield either/or distinctions — the kind of pairs the exam likes to test directly against a scenario.

**Tool versus resource for the same underlying data.** A tool fits when locating or acting on the data requires the model's judgment — a query, a search, an argument-dependent action. A resource fits when the client already knows exactly which piece of data is relevant and can attach it by URI with no model decision involved. If a "resource read" wants a `query` or `filter` parameter, it's really a tool.

**Resource versus prompt.** A resource delivers *data* into context, passively, addressed by URI. A prompt delivers *reusable instruction text*, actively selected, typically by a user rather than the model. Neither performs an action with a side effect the way a tool does.

**Tool, resource, and prompt invocation direction.** A tool is model-initiated. A resource read is typically client-initiated. A prompt is typically user-initiated through client UI. Mixing these up — attributing a resource's client-side attachment to the model's own decision-making, for instance — is a common distractor pattern.

**STDIO versus StreamableHTTP transport.** STDIO: a local subprocess, communicating over stdin/stdout, tied to one client's session, no network exposure, no auth needed beyond machine access. StreamableHTTP: a remote, independently-running server reachable over the network, supporting multiple concurrent clients, requiring real auth, scaling, and audit infrastructure. Transport is a deployment choice, orthogonal to tool definition itself.

**Sampling versus a server's own model access.** Sampling lets a server request an LLM call *through the connecting client*, borrowing its existing model access rather than needing independent credentials. This centralizes cost, governance, and human review at the client rather than scattering unaudited model access across every connected server.

**Notifications versus roots.** Notifications flow server-to-client, signaling a capability or content change. Roots flow client-to-server, declaring the filesystem or resource boundary a server should operate within. Opposite directions, opposite purposes — don't conflate them.

**Progress reporting versus a notification generally.** Progress reporting is a specific, server-to-client update tied to one long-running operation's status. A notification more broadly signals a state change (like an updated tool list) not necessarily tied to any single in-flight call.

**Built-in tool versus MCP-exposed tool.** Identical from the model's perspective — same name/description/schema, same invocation shape. The difference is entirely in who implements and is responsible for what's behind the interface: Claude or the consuming application itself (built-in) versus an external server someone wrote, deployed, and must be vetted (MCP).

**Built-in versus custom, as a decision.** Default to a built-in tool when one already covers the need — the maintenance burden of a redundant custom tool is pure cost. Build custom when the capability is inherently organization-specific, when you need control a built-in tool doesn't expose, or when a built-in tool's scope doesn't match your actual risk tolerance.

**MCP tool design versus ordinary tool design.** There is no separate discipline. Naming, schema structure-versus-prose, granularity, and honest error responses all apply to an MCP-exposed tool exactly as the Tool Layer defined them — MCP only changes the plumbing, and it amplifies the stakes of getting design right because servers are often shared, multi-client infrastructure authored with limited visibility into what else will be in a model's tool set.
