---
title: "Quiz Prep: MCP Server Building (20 Questions)"
parent: MCP Server Building
nav_order: 16
---

# Quiz Prep: MCP Server Building (20 Questions)
{: .no_toc }

Twenty practice questions covering the MCP Server Building module: the three MCP primitives, building a server in Python, tool and resource design under MCP, prompt templates, client implementation, transports, sampling, notifications/roots/progress reporting, and Claude's built-in tools. Work through all twenty before checking the Answer Key at the end.

**1.** What fundamentally distinguishes an MCP tool from an MCP resource?
A. Tools are written in Python and resources are written in JSON.
B. A tool is an invokable action the model decides to call with arguments; a resource is data the client can attach to context by address, with no model decision about whether to retrieve it.
C. Resources always return larger payloads than tools.
D. There is no real distinction; the terms are interchangeable.

**2.** A server exposes individual customer records addressable as `crm://customers/{id}`. A client's user has a specific customer's record already open. What should the client most naturally do to bring that record into context?
A. Have the model call a `search_customers` tool with a guessed query.
B. Read the resource directly by its known URI, with no tool call or model decision involved.
C. Ask the server to email the record to the user.
D. Refuse, since MCP cannot expose individual records.

**3.** Why are MCP prompts used less often in practice than tools or resources?
A. The protocol does not actually support prompts.
B. Prompts require a client to build discovery/selection UI and a human to choose to invoke them, and much of their value overlaps with system-prompt or Skill-level techniques.
C. Prompts are deprecated in the current version of MCP.
D. Prompts can only be invoked by the model, never by a user.

**4.** In a minimal Python MCP server using the high-level decorator-based SDK style, what does a function's docstring primarily become?
A. It is discarded and has no effect.
B. It becomes the tool's description — the primary text the model uses to decide whether and how to call the tool.
C. It becomes the server's transport configuration.
D. It becomes the resource URI scheme.

**5.** In that same decorator-based style, where does a registered tool's input schema come from?
A. A separate, hand-written JSON Schema file the developer must keep in sync manually.
B. The wrapped function's parameter names and type annotations, derived automatically by the decorator.
C. The server's name string.
D. It cannot be generated; every MCP tool must omit a schema.

**6.** A tool function raises `ValueError("days must be between 1 and 7")` when given an out-of-range argument. What Tool Layer principle does this directly reflect, now applied inside an MCP server?
A. Multiple pathways.
B. Designed, descriptive error responses rather than an ambiguous or unhandled failure.
C. MCP governance and server vetting.
D. Resource URI addressing.

**7.** Per this module, how does MCP tool design under the protocol differ from tool design in application code with no MCP involved?
A. It is a wholly separate discipline with its own rules.
B. It doesn't differ in principle — the same naming, schema, and error-response discipline applies — though the stakes are amplified since MCP servers are often shared, multi-client infrastructure with limited visibility into a model's full tool set.
C. MCP tools never need descriptions.
D. MCP tools cannot return errors of any kind.

**8.** What is a concrete, practical reason an MCP tool's name is often qualified with its originating system, such as `jira_create_issue` rather than a bare `create_issue`?
A. It reduces the tool's token cost.
B. It helps avoid name collisions when a client connects multiple independently-authored servers whose tools the author had no visibility into.
C. MCP requires all tool names to contain an underscore.
D. It is purely a cosmetic convention with no functional purpose.

**9.** A "resource read" operation has grown to accept a `filter` and a `sort_by` parameter beyond its identifying URI. What does this most likely indicate?
A. Nothing — resources commonly accept filter parameters.
B. The operation has effectively become a search and should probably be redesigned as a tool instead.
C. The server has a bug in its transport layer.
D. The URI scheme is invalid.

**10.** What is a resource template most useful for?
A. Enumerating a small, fixed set of resources known entirely in advance.
B. Describing a parameterized family of addressable resources (e.g. `tickets://{project}/{id}`) without needing every instance individually listed.
C. Replacing tools entirely.
D. Encrypting resource content.

**11.** What is a client's responsibility during MCP "discovery"?
A. Compiling the server's source code.
B. Requesting the server's list of available tools, resources, and prompts so they can be surfaced to the model or the user appropriately.
C. Deciding which LLM the server is allowed to use.
D. Writing the server's schema definitions on the server's behalf.

**12.** Two independently-authored MCP servers connected to the same client each expose a tool named `search`. Whose responsibility is it to resolve this collision before it reaches the model?
A. The model's, by guessing correctly at inference time.
B. The client's, typically by qualifying tool names with their originating server's identity.
C. Neither — MCP forbids two servers from ever sharing a tool name, so this cannot occur.
D. The end user's, by manually renaming one server's source code.

**13.** Why does STDIO transport require no authentication mechanism of its own?
A. STDIO transport is inherently encrypted.
B. The server runs as a local subprocess with no network listener; reaching it already requires machine-level process access, which is the trust boundary.
C. Authentication is handled by the Python interpreter automatically.
D. STDIO servers do not process any sensitive data by design.

**14.** Which scenario most clearly calls for StreamableHTTP transport rather than STDIO?
A. A single developer's local filesystem-access tool used only in their own editor session.
B. A centrally-hosted server that many teams' applications across an organization need to connect to concurrently.
C. A quick prototype tool tested once on a laptop.
D. A tool that must never be reachable by more than one process.

**15.** Which of the following is a production concern specifically introduced by moving a server from STDIO to a remote, StreamableHTTP deployment?
A. Needing to write a docstring for the first time.
B. Explicit authentication/authorization, concurrent-client scaling, and audit logging, none of which a single-client STDIO subprocess needs to address.
C. The requirement to remove all tools from the server before deployment.
D. The inability to register more than one tool.

**16.** What problem does MCP "sampling" solve?
A. It lets a client bypass a server's authentication requirements.
B. It lets a server request that the connecting client make an LLM call on its behalf, avoiding the need for the server to hold its own independent model credentials.
C. It compresses tool descriptions to save tokens.
D. It allows a server to directly modify the client's own source code.

**17.** Why is it appropriate for a well-behaved client to review or approve a sampling request rather than executing every one automatically?
A. Because sampling requests are always malformed.
B. Because a sampling request effectively spends the user's or organization's own model usage on the server's behalf, and unchecked compliance would let any connected server consume that usage without oversight.
C. Because sampling is not part of the official MCP specification.
D. Because clients are technically incapable of making LLM calls.

**18.** A server notifies a connected client that its list of available tools has changed mid-session. What should a well-behaved client do?
A. Ignore the notification and continue operating on the tool list fetched at connection time.
B. Re-run discovery so the model's available tool set reflects the server's current capabilities rather than a stale snapshot.
C. Immediately disconnect from the server permanently.
D. Forward the raw notification text to the model as a tool result.

**19.** A client declares a "root" of `/home/user/project` when connecting a filesystem MCP server. What is this root communicating?
A. The exact tool the server must call next.
B. The boundary within which the server should operate, extending the same least-privilege scoping principle covered in the Tool Layer's governance lecture.
C. The model's confidence score for its next action.
D. The server's own internal file structure.

**20.** An organization already has Claude's built-in web search available and is deciding whether to build a custom web-fetching tool instead. Per this module's decision framework, what should drive that decision?
A. Custom tools should always be preferred over built-in ones as a matter of principle.
B. Whether there's a specific, articulable gap — missing control, an unmet requirement, or a capability the built-in tool genuinely lacks — since a redundant custom tool otherwise just adds unnecessary ongoing maintenance burden.
C. Whichever option uses fewer tokens per call, with no other consideration.
D. Built-in tools cannot be used alongside any custom tools in the same session.

## Answer Key

**1. B** — A tool is model-invoked with arguments and potential side effects; a resource is client-attached, read-only data addressed by URI, with no model decision about the retrieval itself.

**2. B** — Since the client already knows exactly which record is relevant, reading the resource directly by its known URI is the natural fit — no tool call or model judgment is needed.

**3. B** — Prompts depend on client-built discovery/selection UI and a human choosing to invoke them, and much of what they'd provide overlaps with system-prompt- and Skill-level techniques covered elsewhere in the course.

**4. B** — In the decorator-based style, the docstring becomes the tool's description, the primary signal the model uses to decide whether and how to call it.

**5. B** — The decorator derives the input schema automatically from the function's parameter names and type annotations rather than requiring a separately hand-written schema.

**6. B** — Raising a clear, descriptive exception on invalid input is exactly the designed-error-response principle from the Tool Layer, now shown inside an MCP tool's implementation.

**7. B** — MCP tool design is not a separate discipline; the same principles apply, just with amplified stakes given MCP servers' typically shared, multi-client nature and reduced visibility into a model's full tool set.

**8. B** — Qualifying names with the originating system's identity is a practical mitigation against name collisions across independently-authored, simultaneously-connected servers.

**9. B** — Wanting filter/sort parameters on a "resource read" is the tell that the operation actually requires query-like reasoning and should be redesigned as a tool.

**10. B** — A resource template describes an entire parameterized family of addressable resources without requiring each instance to be individually enumerated in advance.

**11. B** — Discovery is the client requesting the server's lists of tools, resources, and prompts so they can be correctly surfaced downstream.

**12. B** — Resolving name collisions across connected servers — commonly by qualifying names with server identity — is the client's responsibility.

**13. B** — STDIO has no network listener at all; reaching a STDIO server already requires machine-level process access, which serves as its implicit trust boundary.

**14. B** — Shared, concurrently-accessed organizational infrastructure is exactly the case StreamableHTTP is built for, unlike STDIO's single-client subprocess model.

**15. B** — Explicit auth, concurrent-client scaling, and audit logging are production concerns a single-client STDIO subprocess simply doesn't face, since it has no network exposure or concurrent-client reality to begin with.

**16. B** — Sampling lets a server borrow the connecting client's existing model access for an LLM call it needs, rather than requiring its own independent credentials.

**17. B** — A sampling request spends the user's or organization's own model usage; client-side review keeps that usage subject to the same oversight as any other consequential action, rather than letting any connected server consume it unchecked.

**18. B** — A well-behaved client re-runs discovery on a capability-change notification rather than continuing to operate on a stale tool list.

**19. B** — A root declares the operating boundary (typically a filesystem scope) a server should respect, extending least-privilege scoping to the filesystem-boundary case specifically.

**20. B** — The decision should turn on a specific, articulable gap the built-in tool doesn't cover; absent that, a custom equivalent just duplicates maintenance burden for no real gain.

<!-- prevnext:start -->

---

| [&larr; Previous: Key Distinctions - MCP Server Building](15-key-distinctions-mcp-server-building) | [Next: Claude Code Deep Dive &rarr;](../17-claude-code-deep-dive/) |
|:---|---:|

<!-- prevnext:end -->
