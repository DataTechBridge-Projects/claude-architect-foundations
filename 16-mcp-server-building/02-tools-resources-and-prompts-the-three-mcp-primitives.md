---
title: "Tools, Resources & Prompts: The Three MCP Primitives"
parent: MCP Server Building
nav_order: 2
duration: "15:00"
---

# Tools, Resources & Prompts: The Three MCP Primitives
{: .no_toc }

*Video duration: 15:00*

The MCP Architecture & Governance lecture in the Tool Layer named all three primitives in a single sentence and moved on, because that lecture's job was the protocol's shape, not its parts. This lecture takes each of the three — **tools**, **resources**, and **prompts** — and gives it the depth it deserves, because a server author reaches for a specific one of these three shapes every time they decide how to expose a new piece of a system, and choosing wrong produces exactly the kind of friction the rest of this module works to avoid.

## Tools: invokable actions

A **tool**, in MCP, is structurally identical to every tool definition covered in the Tool Layer: a name, a human- and model-readable description, and an input schema describing the arguments it accepts. What makes something a *tool* rather than a resource or a prompt is **not its shape but its nature**: a tool represents an action with a side effect, or at minimum a computation the model has to explicitly request that returns a result specific to the arguments it was invoked with. Creating a calendar event, sending a Slack message, writing a database record, searching an index with a particular query, restarting a service — these are all things that happen because the model decided, in a specific turn, that they should happen, with specific arguments determining exactly what occurs.

This is the primitive you already know the most about, because everything the Tool Layer taught about naming discipline, schema design, and error-response quality applies to an MCP-exposed tool with zero modification. An MCP server author, in the moment they write a tool's description and schema, is doing exactly the same job as an engineer writing a tool directly into an application's code — the only difference is that the resulting definition gets exposed through the protocol's standard listing-and-invocation mechanism instead of being wired in by hand. A `create_ticket` tool exposed by a Jira MCP server needs the same disambiguating description, the same well-chosen granularity, and the same honest, retryable-or-not error response as a hand-rolled `create_ticket` tool would need in an application with no MCP involved at all.

## Resources: readable data without an action

A **resource** is data the client can retrieve and pull into the model's context — a file's contents, a database record, a configuration value, a log excerpt — without that retrieval constituting an action with side effects. Each resource is identified by a **URI**, a unique address the server assigns it, and a client reads a resource roughly the way a browser fetches a page: by asking for that URI and getting content back, with no side effect implied by the read itself beyond the read.

The distinction that matters most here is the one between a resource and a tool that happens to return read-only data, because on the surface they can look similar — both end up putting information into context. The difference is architectural rather than superficial: a resource is something the *client* — not the model — can decide to attach to context, often ahead of time or in response to a user action, without needing the model to reason about calling anything at all. A file open in an editor, a document a user has pinned to a conversation, a project's README — these are naturally resources, because the decision to include them doesn't need to run through the model's judgment about whether an action should be taken. A tool, by contrast, requires the model to actively decide, mid-inference, that a specific invocation with specific arguments should happen right now. If retrieving a piece of data never needs the model's judgment about *whether* and *with what arguments* to retrieve it — if the answer is simply "the client already knows this is relevant" — that's the signature of a resource, not a tool.

{: .important }
> Don't confuse a resource with "a tool that happens to return read-only data." The test is who decides: if the client already knows which piece of data is relevant and can attach it by URI, it's a resource; if locating or acting on the data requires the model's judgment mid-reasoning, it's a tool — even when both end up putting identical content into context.

Resources also typically support **listing** — a server can expose an enumerable set of resources it holds (all the files in a directory, all the tickets in a project) so a client can present that set to a user, or in some implementations to the model, without every item's URI needing to be known in advance. This listing capability is itself one of the practical reasons resources exist as a distinct primitive: a tool call each time you wanted to know what data is even available would be a clumsy way to expose an entire browsable data set that a client might reasonably want to page through before ever touching the model.

## Prompts: reusable templates

A **prompt**, in MCP's specific sense, is a server-defined, parameterized template for a piece of text the server expects to be genuinely useful as a starting point for a model interaction — not a system prompt in the sense of "instructions governing an entire session," but a reusable, fill-in-the-blank chunk of prompt content a server offers, typically surfaced to a user as a selectable option (often through a client's UI, like a slash command or a menu item) rather than something the model decides to invoke mid-reasoning the way it would a tool.

A code-review MCP server might expose a `review-pull-request` prompt template that takes a PR identifier as a parameter and expands into a full, carefully-worded instruction — "Review the following pull request for correctness, style, and test coverage, flagging any breaking changes explicitly..." — so that any client connecting to that server gets a consistent, well-tuned starting point for that recurring task rather than every user or every application re-inventing that wording from scratch. The value proposition is the same one behind reusable prompt libraries generally: the server author, who understands the underlying system and the task best, captures that expertise once as a template, and every client benefits from it without needing to have separately developed the same wording.

## Why prompts see less use in practice

Of the three primitives, prompts are consistently the least used in real MCP deployments, and it's worth understanding why rather than treating it as an arbitrary fact. Tools and resources both plug directly into the model's own reasoning loop — a tool gets called because the model decided an action was needed; a resource gets read because the client determined it was relevant context — and in both cases, the primitive is doing work inside a flow the model and client are already engaged in. A prompt, by contrast, is typically surfaced as something a *user* selects deliberately, which means it depends on a client actually building UI around prompt discovery and selection, and it depends on users knowing the template exists and choosing to invoke it. Many clients, including some widely used ones, have historically implemented tools and resources fully while treating prompt support as optional or partial, which further limits how much a server author can rely on prompts being available at all. The practical result is that most server authors invest their design effort in tools first, resources second, and prompts only when there's a genuinely recurring, well-defined task worth templating and a client ecosystem that will actually surface it.

## A worked example: one system, three primitives

Consider an MCP server built around a company's internal documentation wiki. As **tools**, it might expose `search_docs` (query the wiki's search index and return matching page titles and snippets) and `create_page` (write a new wiki page given a title and body) — both invokable actions with arguments and results, both benefiting from the same naming and schema discipline as any other tool. As **resources**, it might expose each individual wiki page by a `wiki://` URI, so that a client whose user has a specific page open, or has pasted a link to one, can pull that page's full content into context directly, without the model needing to call `search_docs` and guess at a query to relocate a document the user already identified. As a **prompt**, it might offer a `draft-release-notes` template that takes a version number and a date range, expanding into a fully-worded instruction to search recent pages tagged as changes within that range and synthesize them into release notes — a recurring, well-defined task worth capturing once rather than re-prompting from scratch every time someone needs it.

Notice that the tool and the resource in this example touch overlapping data — both can surface a wiki page's content — but they exist for different reasons and get reached for under different circumstances. `search_docs` earns its place because the model doesn't yet know *which* page it needs; the resource earns its place because **the client, not the model,** already knows exactly which page is relevant and doesn't want to spend a full tool-call round trip re-deriving that.

## How the three relate

It's useful to hold all three together rather than picking whichever seems most familiar. The clearest way to keep them straight: a **tool** is something the model decides to invoke, with arguments, expecting a result and possibly a side effect. A **resource** is something the client can attach to context by URI, without needing the model to decide anything about the retrieval itself. A **prompt** is a reusable, parameterized starting point for an interaction, typically chosen by a user rather than by the model mid-reasoning. The same underlying system frequently exposes more than one — a filesystem-oriented MCP server might expose `read_file` and `write_file` as tools, individual files as resources addressable by a `file://` URI, and a `summarize-directory` prompt template as a convenience — and choosing which primitive fits a given piece of functionality is itself a design decision with the same stakes as the tool-granularity questions from the Tool Layer.

| Primitive | Who decides to use it | What it delivers | Side effects |
|---|---|---|---|
| Tool | The model, mid-reasoning, with arguments | An action's result | Possible |
| Resource | The client, by attaching a known URI | Readable context data | None |
| Prompt | The user, via client UI (menu, slash command) | A reusable, parameterized instruction template | None |

**Why it matters for the exam:** Expect direct scenario questions asking you to identify which of the three primitives best fits a described piece of functionality — particularly the tool-versus-resource distinction, since that's where real ambiguity exists and where the exam is most likely to test judgment rather than rote recall. The test to apply is always the same: does using this piece of functionality require the model to decide, mid-reasoning, to invoke something with specific arguments (a tool), or can the client simply attach it to context without that decision (a resource)? And is it a reusable template a user selects rather than the model (a prompt)?

<!-- prevnext:start -->

---

| [&larr; Previous: Module Intro](01-module-intro) | [Next: Building an MCP Server in Python &rarr;](03-building-an-mcp-server-in-python) |
|:---|---:|

<!-- prevnext:end -->
