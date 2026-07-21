---
title: Tool Definition & Selection
parent: The Tool Layer
nav_order: 2
duration: "18:45"
---

# Tool Definition & Selection
{: .no_toc }

*Video duration: 18:45*

The Agentic Layer's lecture on structured tool use established the mechanics: a tool is a name, a description, and a JSON Schema, sent to the model alongside every request, and the model responds with a structured call rather than free-form prose. This lecture is about the layer above that mechanism — not *whether* tools are structured, but *how well-designed* a given structured tool actually is. Two tools can have identical, perfectly valid JSON Schemas and wildly different real-world reliability, because the schema only constrains the shape of a valid call. It says nothing about whether the model picks the right tool in the first place, or fills in that shape correctly. That's a design problem, not a validation problem, and it's the one this lecture addresses head-on.

## Tool design is API design for a different caller

Start from an analogy you likely already have working instincts for. When you design a REST endpoint or a library function for other engineers, you think about naming conventions, parameter clarity, sensible defaults, and documentation that answers the questions a caller will actually have. Good API design isn't a matter of taste — it's a matter of reducing the number of ways a caller can misuse the interface, and making the interface's intended use obvious from its shape alone.

Tool design for a language model is the same discipline, aimed at a different kind of caller. A human engineer reading API documentation can tolerate real ambiguity: they can check a second source, ask a colleague, read the implementation, or simply try something and observe what happens in a low-stakes sandbox before committing to a production call. A model deciding which tool to invoke mid-inference has none of that. It has the tool's name, its description, its schema, and whatever's currently in its context — and it has to decide, in that moment, which tool (if any) to call and what arguments to pass. There's no out-of-band clarification available unless you've explicitly built one in. Every ambiguity you leave in a tool definition becomes, statistically, a wrong call some non-trivial fraction of the time.

This is why tool design deserves to be treated as its own discipline rather than an afterthought bolted onto a function you already wrote for some other purpose. The function's internal implementation can be arbitrarily complex; what the model sees is only the name, the description, and the schema, and *that surface* is what determines whether the tool gets used correctly. A tool with excellent implementation but a vague description will be selected incorrectly or under-selected regardless of how well it works once called. A tool with a mediocre implementation but a crystal-clear description, tight schema, and honest error messages will often outperform it in practice, because the model reaches for it confidently and uses it correctly.

## Anatomy of a tool definition, revisited with intent

Recall the shape from the Agentic Layer: a `name`, a `description`, and an `input_schema`. Each of these three fields carries distinct design weight, and it's worth taking them one at a time with selection — not just validity — as the goal.

### Naming

A tool's name is the first and cheapest signal the model uses to disambiguate among the available options, often before it even reads the full description closely, especially once a tool set grows past a handful of entries. Good names are specific, action-oriented, and consistent in convention across the tool set. `search_issues` is a better name than `query` or `handle_request`, because it tells the model both the domain (issues) and the verb (search) without requiring it to read further. Consistency matters just as much as specificity: if one tool is `get_user`, a sibling tool for looking up an order should be `get_order`, not `order_lookup` or `fetch_order_details`. Inconsistent naming conventions across a tool set force the model to learn each tool's idiosyncratic vocabulary individually rather than pattern-matching on a convention, which is exactly the kind of avoidable friction that produces wrong selections under load.

Names should also avoid deceptive generality. A tool named `update_record` that in practice only updates one specific type of record in one specific system invites misuse — the model may reasonably assume it applies more broadly than it does, because the name told it so. If a tool is narrow, its name should say so: `update_invoice_status` rather than `update_record`.

### Description

The description is where most of the real selection-quality work happens, and it's the field most commonly under-invested in. A weak description states only what the tool does: "Searches issues." A strong description states what it does, *when to use it relative to alternatives*, and any constraints or side effects the model needs to know before calling it: "Search the issue tracker for issues matching a query. Use this before creating a new issue, to check for duplicates. Does not search closed issues by default — pass status: 'all' to include them."

That second sentence is doing real disambiguation work if there's also a `create_issue` tool in the same tool set — it's actively steering the model toward a sequencing decision (search first, then create) that the schema alone could never express. This is the single most important thing to internalize about description writing: a description isn't documentation of the tool in isolation, it's guidance for choosing correctly *among the full set of tools available in that context*. You should write — and revise — tool descriptions with the entire tool set in view, not tool by tool in isolation, precisely because selection errors are relative: the model isn't asking "is this tool good," it's asking "is this tool the *right one, right now, compared to its neighbors*."

Descriptions should also be honest about limitations and edge cases the model would otherwise discover only by trial and error: rate limits, size limits, what happens with an empty result, whether an action is destructive or reversible. A model that doesn't know a `delete_file` tool is irreversible will call it with the same confidence it calls a harmless read tool, because nothing in the definition told it otherwise. This connects directly to workflow safeguarding from the Agentic Layer — the tool's own description is one of the places blast-radius information can and should live, in addition to whatever gating exists in the surrounding system.

### Input schema

The schema is where structure does its enforcement work: types, required versus optional fields, enums for constrained value sets, minimums and maximums for numeric ranges. Beyond validity, a well-designed schema also *teaches* the model how to use the tool correctly, because the model reads field names and per-field descriptions as part of forming its call. A field named `q` with no description teaches the model nothing; a field named `query` with a description like "Search terms, supports quoted phrases for exact match" teaches the model both what to put there and a capability (quoted-phrase search) it might not have otherwise inferred was available. Field-level descriptions are frequently skipped by engineers moving quickly, and it's a real cost — every under-documented field is a place where the model is guessing rather than being told.

Enums deserve particular attention because they do double duty: they constrain the schema for validation *and* they enumerate the valid options directly in the tool definition the model sees on every call, functioning as a built-in reference the model doesn't have to infer or remember from earlier in the conversation. A `status` field with `"enum": ["open", "closed", "all"]` tells the model exactly what's valid without any prose explanation required. Where a field's legitimate values are genuinely enumerable, prefer an enum over a free-text string with a description saying "must be one of open, closed, or all" — the enum is machine-enforced and can't be silently violated by a slightly-off string like `"Open"` or `"active"`.

## The core selection problem: similar tools, ambiguous boundaries

Everything above matters most in exactly the situation that causes the most real-world selection failures: a tool set with several tools that are plausibly relevant to the same request. Consider a system with `search_issues`, `search_pull_requests`, and `search_all`. A user asks the model to "find anything related to the login bug." Which tool should the model call? Without careful description writing, this is genuinely ambiguous — "anything" could reasonably map to any of the three.

The fix isn't cleverness in the model — it's precision in the descriptions, written with the other tools explicitly in mind: `search_issues` should state that it only searches issues, not pull requests; `search_all` should state that it's the right choice specifically for exploratory or cross-cutting queries where the type of match isn't known in advance. Once the descriptions carve up the space of "when do I use this one, versus that one" explicitly, the ambiguity mostly disappears — not because the model got smarter, but because the disambiguating information that used to live only in an engineer's head now lives in the definitions themselves.

This is a recurring theme worth naming directly: **tool selection failures are usually definition failures, not model failures.** When a model calls the wrong tool among several similar options, the productive response is almost never "the model should have known better" — it's "what information did the model actually have available to distinguish these, and where was it missing?" Debugging a bad tool call the same way you'd debug a bad function call — by reading the definition as if you'd never seen the system before — is a genuinely useful practice, and one the exam rewards you for reasoning through explicitly.

## The granularity tradeoff: too many tools versus too few

Underneath the naming-and-description craft sits a structural decision that shapes everything else: how many tools should exist, and how narrowly should each one be scoped? This tradeoff pulls in two directions, and both extremes fail in characteristic, recognizable ways.

**Too many overlapping tools** — a `search_issues`, a `search_issues_by_author`, a `search_issues_by_label`, a `search_recent_issues`, each a thin variant of the others — multiplies the disambiguation burden discussed above. Every new tool added to a set is a new boundary the model has to correctly reason about relative to every existing tool, and that burden grows faster than the tool count itself, because it's roughly a pairwise comparison problem. A large tool set of narrow, overlapping tools tends to produce more selection errors, not fewer, even though each individual tool is simpler to implement and document than one general tool would be. It also inflates the token cost of every single model call, since tool definitions are sent on every request regardless of whether they end up used — a cost that's easy to overlook when adding "just one more" narrow tool feels cheap in isolation.

**Too few, overloaded tools** — a single `search` tool with a dozen optional parameters covering issues, pull requests, comments, users, and commits, each interacting with several of the others in tool-specific ways — pushes the disambiguation burden in the opposite direction. Instead of choosing among tools, the model now has to correctly navigate a large, tangled parameter space within one tool, and get the *combination* of parameters right, which is often harder to specify unambiguously in a single description than splitting the tool would have been. Overloaded tools also tend to produce more subtly wrong calls — technically schema-valid, but a combination of parameters that doesn't actually do what the caller intended — because the schema can enforce individual field validity but rarely enforces the more complex cross-field constraints that would catch these.

| Aspect | Too many overlapping tools | Too few, overloaded tools |
|---|---|---|
| Failure surfaces as | Wrong tool picked among near-duplicates | Wrong parameter combination within one tool |
| Burden on the model | Repeated pairwise choice among tools | Navigating a tangled, cross-field parameter space |
| Cost as the set grows | Grows faster than tool count (roughly pairwise) | Grows with schema complexity, not tool count |
| Token cost per call | Every tool definition sent on every request | One definition, but a larger schema |
| Typical fix | Merge near-duplicates into fewer, sharply differentiated tools | Split along the one dimension that's usually obvious at call time |

The resolution isn't a fixed number of tools — it's matching granularity to how *distinct* the underlying operations genuinely are, and how often they're actually invoked in combination. If two operations differ in exactly one dimension that the model reliably has available at call time (searching issues versus searching pull requests, where the "which entity type" decision is usually obvious from the user's request), separate tools with narrow, sharply differentiated descriptions typically outperform one overloaded tool. If the "operations" are really just filters on one coherent underlying action (search, with an optional entity-type filter, an optional author filter, an optional date range), a single well-schemed tool with clearly described optional parameters is usually easier for the model to use correctly than three-to-five near-duplicate tools would be, because there's no selection step required at all — only a parameter-filling step, which schemas are specifically good at constraining.

A useful diagnostic question when you're unsure which side of this line a tool set falls on: *would a competent human API consumer, reading only the names and descriptions with no other context, be able to say confidently which one to call for a given request, every time?* If the answer is "usually, but there are a couple of cases that would genuinely require guessing," you've found exactly the ambiguity a model will also stumble on — and it's cheaper to fix at design time than to discover through a stream of subtly wrong tool calls in production.

## Naming and description as a living interface, not a one-time artifact

One last point worth carrying forward into the rest of this module: tool definitions aren't "done" once they pass schema validation and the happy path works in testing. Selection failures typically surface only once a tool set grows, once real user phrasing turns out to be more varied than whatever you tested with, or once a new, similar tool is added later and creates an ambiguity that didn't exist when the original tool was designed in isolation. Treating tool descriptions as a living interface — something you revisit and sharpen as you observe real selection behavior, the same way you'd revise API documentation after watching real integration partners misuse it — is part of what separates a tool set that degrades gracefully as it grows from one that becomes progressively harder for the model to navigate reliably.

**Why it matters for the exam:** Expect scenario questions that present two or three tool definitions with overlapping descriptions and ask which one the model would most plausibly select, or misselect, for a given request — you should be able to point to the exact word or missing detail responsible. You should also be able to argue, from first principles, whether a described tool set is too fragmented or too overloaded, and what a better granularity boundary would look like, since that judgment call is one of this domain's most frequently tested distinctions.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Intro](01-module-intro) | [Next: Structure with Prose as the Fallback &rarr;](03-structure-with-prose-as-the-fallback) |
|:---|---:|

<!-- prevnext:end -->
