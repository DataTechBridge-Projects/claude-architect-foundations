---
title: Defining and Exposing Resources
parent: MCP Server Building
nav_order: 5
duration: "12:00"
---

# Defining and Exposing Resources
{: .no_toc }

*Read duration: 12:00*

The previous primitives lecture drew the resource-versus-tool line at a conceptual level: a resource is something a client attaches to context by address, without the model needing to decide anything about the retrieval itself. This lecture goes deeper into what that actually looks like to build — URIs, how a client reads a resource in practice, and a sharper version of the "resource or tool" decision for cases that genuinely could go either way.

## URIs: how a resource gets addressed

Every resource an MCP server exposes is identified by a **URI** — a unique string address, following the same general syntax family as a web URL, that the server assigns and that a client uses to request that specific resource's content. A filesystem-oriented server might address files as `file:///home/user/project/README.md`; a server wrapping an internal ticketing system might use something like `tickets://PROJ-4521`; a database-backed server might address a specific record as `db://customers/48213`. The scheme portion (`file://`, `tickets://`, `db://`) signals what kind of resource this is and, often, which server or subsystem it belongs to, while the remainder of the URI pins down the specific instance.

This addressing scheme is what makes resources genuinely composable across a session in a way an ad hoc "fetch this data" tool call isn't. Once a client has a resource's URI — because a user pasted a link, because a previous tool call returned one, because a resource listing enumerated it — that URI is a stable, reusable handle. The same document can be referenced, re-read, or handed to a different part of a client's UI without needing to re-run whatever process originally produced it, the same way a URL is a durable handle to a web page regardless of how you first arrived at it.

## Reading a resource: the client's side

A client reads a resource through a dedicated protocol operation distinct from a tool call: it sends the resource's URI to the server and receives that resource's content back — typically text, though the protocol accommodates binary content (images, for instance) where appropriate. Critically, this operation carries none of the semantics of a tool invocation: there's no notion of the model having "decided" to call anything, no arguments beyond the URI itself to reason about filling in correctly, and no side-effect risk the way an action-oriented tool call carries. This is precisely why a client, rather than the model, is typically the party that decides to read a resource and attach it to context — the operation is cheap enough, and side-effect-free enough, that **it doesn't need to be gated behind the model's judgment** the way an action does.

Servers also commonly expose a **resource listing** operation, letting a client enumerate what resources exist without needing every URI known in advance — list the files in a directory, list the open tickets in a project, list the tables in a database. A client's UI might use this listing to let a user browse and select a resource to attach (a file picker built directly against a connected filesystem server, for instance), which is a distinctly client-side, often pre-model-involvement interaction that has no real equivalent among tools.

## Resource templates: parameterized addresses

Rather than enumerating every possible resource URI a server could ever expose, a server can instead expose a **resource template** — a URI pattern with placeholder segments, such as `tickets://{project}/{ticket_id}`, that describes an entire addressable family of resources without listing each one individually. A client (or, in some implementations, a model reasoning about what URI to construct) can fill in the placeholders to address a specific resource that was never explicitly enumerated, which matters most when the space of possible resources is large, dynamic, or effectively unbounded — every ticket that could ever exist in a live ticketing system, for instance, rather than a small, fixed set of files.

## Resource versus tool for the same underlying data: the sharper version

The primitives lecture gave you the conceptual test — does retrieving this data require the model's judgment about whether and how to retrieve it? This lecture adds the practically sharper version, because plenty of real cases sit close enough to the line that the conceptual test alone doesn't immediately resolve them.

Lean toward a **resource** when: the data has a stable, known, or enumerable address independent of any particular query (a specific file, a specific record, a specific document); **the client — not the model — is the party positioned to know** which piece of data is relevant (a file a user has open, a document a user just linked); and reading it carries no side effect and needs no argument beyond "which one."

Lean toward a **tool** when: locating the right data requires a query, a search, or reasoning that only the model is positioned to perform (finding *which* tickets match a natural-language description of a bug, rather than reading one you already know the ID of); the operation might reasonably fail in ways that need a structured, retryable-aware error response; or the same underlying capability needs to support filtering, sorting, or other parameters beyond a bare identifier.

A genuinely useful heuristic: if you find yourself wanting to add a `query` or `filter` parameter to a "resource read," that's usually a sign the operation has drifted into being a search — a tool's job — dressed up as a resource. A resource's only real parameter, once a template's placeholders are filled in, should be "which specific one," not "which ones, matching what criteria."

{: .important }
> If a "resource read" needs a `query` or `filter` parameter, that's the signal it's actually a **tool wearing a resource's clothes** — a resource's only legitimate parameter is "which specific one," never "which ones, matching what criteria."

| Signal | Resource | Tool |
|---|---|---|
| Locating the right data | Client already knows the address | Requires a query, search, or model judgment |
| Extra arguments beyond identity | None | Filter, query, or other parameters |
| Side effects | None | Possible |
| Failure handling | Simple read, no result to negotiate | Structured, retryable-aware error response |

## Subscriptions: staying current on a resource that changes

A resource's content isn't necessarily static for the life of a connection, and MCP accounts for this with **subscriptions**: a client can subscribe to a specific resource, and the server notifies it when that resource's underlying content changes, rather than the client needing to re-read it repeatedly on some arbitrary polling schedule to catch updates. A client with a log file attached as a resource, for instance, can subscribe to it and receive an update signal each time new lines are appended, keeping whatever's in context current without wasteful, blind re-fetching. This is the same underlying notification mechanism covered later in this module applied specifically to resource freshness — another example of MCP's supporting mechanics existing to keep a long-lived session honest about state that can change out from under it.

## A worked example: a filesystem server's resource design

Consider a filesystem-oriented MCP server. Each file is a natural resource, addressable as `file:///path/to/file.py`, read on demand and requiring no query or judgment call to locate once the client already knows the path — a user opening a specific file in an editor is the clearest possible case for a resource rather than a tool. Contrast this with the same server's `search_files` tool, which takes a pattern or a natural-language description and returns a list of matching paths: locating *which* files match an underspecified criterion is exactly the kind of judgment call that belongs to the model, mediated through a tool, not something a client can resolve merely by already knowing an address. The same server frequently offers both, and a well-designed one is deliberate about not blurring them — `search_files` shouldn't quietly also accept a specific known path and skip the search logic entirely, since that conflates two operations with genuinely different shapes and different callers (the model deciding what to search for, versus the client already knowing exactly what it wants) into one.

## Binary and structured resource content

Not every resource is plain text. A resource can return binary content — an image, a PDF, an audio clip — appropriately encoded for transport, or structured data such as JSON, when that's the more faithful representation of what the resource actually is. A `db://customers/48213` resource might reasonably return a structured JSON record rather than a prose paragraph describing that customer, since the underlying data is naturally structured and a client or a downstream process may want to consume it as such rather than parse prose back apart. Choosing the right content type for a resource is a smaller-scale version of the structure-versus-prose tradeoff the Tool Layer raised for tool schemas: pick the representation that's most faithful to what the data actually is, not the one that's most convenient to hard-code.

**Why it matters for the exam:** Expect scenario questions that describe a piece of functionality and ask you to identify it as a resource, a resource template, or a tool disguised as one. The reliable discriminator the exam is testing is whether the operation needs a query or search-like reasoning (tool) versus a stable, already-known address (resource) — and secondarily, whether the client or the model is the party actually positioned to decide what's relevant.

<!-- prevnext:start -->

---

| [&larr; Previous: Defining and Exposing Tools via MCP](04-defining-and-exposing-tools-via-mcp) | [Next: MCP Prompt Templates &rarr;](06-mcp-prompt-templates) |
|:---|---:|

<!-- prevnext:end -->
