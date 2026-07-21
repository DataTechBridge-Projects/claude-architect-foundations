---
title: MCP Prompt Templates
parent: MCP Server Building
nav_order: 6
duration: "10:00"
---

# MCP Prompt Templates
{: .no_toc }

*Video duration: 10:00*

Tools and resources have both had a full lecture apiece; prompts get a shorter one, and that asymmetry is itself informative rather than accidental — the primitives lecture already flagged prompts as the least-used of the three, and this lecture explains that gap in enough depth to reason about it confidently on the exam, alongside what a prompt template actually is and where it genuinely earns its place.

## What a prompt template actually is

An MCP **prompt** is a server-defined, named, parameterized piece of prompt text — a template with placeholders a caller fills in, which the server expands into a complete, ready-to-use instruction. Unlike a tool, invoking a prompt doesn't perform an action against an external system or return data retrieved from one; it returns *text meant to become part of a model interaction*, typically inserted as if the user (or the client, on the user's behalf) had written it themselves. A prompt is best understood as a reusable macro for a recurring, well-articulated task, authored once by whoever understands both the underlying system and the task well, and made available to every client that connects to that server.

A concrete example: a database MCP server might expose an `explain-slow-query` prompt template taking a query string as its parameter, expanding into something like "The following SQL query is running slower than expected: `{query}`. Analyze it for missing indexes, inefficient joins, or unnecessary full table scans, and propose a specific optimization." A user (or a client's UI) invoking that template gets a carefully-worded, expert-crafted starting point for a recurring diagnostic task, without needing to have written that framing themselves, and without every team independently re-deriving similarly-good wording from scratch.

## How a prompt gets invoked

Because a prompt isn't something the model decides to call mid-reasoning the way it would a tool, its invocation path typically runs through the client's own interface rather than the model's tool-use loop: a user selects a named prompt from a menu, slash-command list, or similar affordance a client builds around prompt discovery, optionally supplies the template's parameters, and the client sends the resulting expanded text into the conversation as though it were user-authored input. This is a meaningfully different invocation path from either a tool (model-initiated, argument-filled by the model, executed by the server) or a resource (client-initiated, addressed by URI, attached silently to context) — a prompt is deliberately the primitive most oriented around a human's explicit choice to use it.

## Use cases where prompts genuinely earn their place

Prompts fit best where a task recurs often enough to be worth templating, is well-defined enough that a fixed structure with a few parameters captures it faithfully, and benefits from wording expertise the typical end user doesn't have and shouldn't need to reconstruct each time. A code-review server's `review-pull-request` template, a support-ticket server's `draft-customer-response` template given a ticket ID and desired tone, or a documentation server's `generate-changelog` template given a version range are all reasonable fits: each captures a recurring, nameable task behind a small parameter set, and each represents real authoring effort — knowing what to ask for and how to phrase it — that's genuinely worth not re-doing per user, per session.

## Why prompts see less use than tools or resources

Three structural reasons compound here, and understanding all three matters more than memorizing any one. First, discoverability depends entirely on client support: a prompt is only useful if the connected client builds real UI around browsing and invoking named prompts, and that UI investment has historically lagged behind tool and resource support across the client ecosystem, since tools and resources are load-bearing for basic agentic function in a way prompts simply aren't. Second, prompts require a human in the loop choosing to invoke them, which caps their usefulness in fully autonomous or headless agentic workflows where nobody is browsing a menu at all — an unattended agent has no natural moment at which a prompt template gets selected the way a tool gets called or a resource gets attached. Third, a large share of what a prompt template would do — expand a short instruction into a fuller, better-worded one — can often be accomplished just as well by putting that same expertise directly into a system prompt, a Skill, or a CLAUDE.md convention (concepts covered in the Prompt and Workflow Layers), which compete directly with MCP prompts for the same underlying need without requiring a dedicated server connection or client-side prompt-discovery UI at all.

The practical upshot is that prompts occupy a real but narrow niche: genuinely useful when a server author wants a recurring, well-defined task's wording to travel with the server itself, to any client that connects, rather than being redefined per application — but not a primitive most server authors reach for first, and not one the exam expects you to over-weight relative to tools and resources.

## Prompts versus a tool that "just returns text"

It's worth being precise about why a prompt template isn't simply a tool that happens to return prompt-shaped text, because the two can look superficially similar. A tool the model calls to generate boilerplate text still runs through the model's own tool-selection judgment — the model has to decide, mid-reasoning, that calling it is the right move, exactly like any other tool. A prompt template, by contrast, is selected before the model's reasoning about this particular task even begins — a user (or a client on the user's behalf) chooses it as the starting frame for the interaction, and the expanded text becomes input the model then reasons over, not a result the model requested mid-task. The direction of control is inverted: a tool is something the model reaches for during its own reasoning; a prompt is something that shapes what the model is asked to reason about in the first place.

## Designing a prompt template well

A well-designed prompt template shares real DNA with a well-designed tool schema, even though the two primitives serve different purposes. Its parameters should be as constrained as the task allows — a `tone` parameter for a customer-response template is better served by an enum (`formal`, `friendly`, `apologetic`) than an open free-text field, for exactly the structure-over-prose reasons the Tool Layer established for schemas generally. Its expanded text should be specific enough to actually save the effort of writing the instruction from scratch, not so generic that a user gains little over just typing their own request directly — a template that expands to "Please help with the following: {input}" isn't earning its place as a distinct primitive at all, since it adds a discovery and invocation step without adding any actual authored value.

**Why it matters for the exam:** Expect the exam to test that you understand *why* prompts are used less, not just that they are — client UI investment lagging behind tools and resources, the human-in-the-loop requirement that doesn't fit headless workflows, and overlap with system-prompt-level techniques covered elsewhere in the course are all fair game as the "correct reason" among plausible-sounding distractors.
