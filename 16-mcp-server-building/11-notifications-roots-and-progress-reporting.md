---
title: Notifications, Roots, and Progress Reporting
parent: MCP Server Building
nav_order: 11
duration: "11:00"
---

# Notifications, Roots, and Progress Reporting
{: .no_toc }

*Video duration: 11:00*

Tools, resources, prompts, and sampling are MCP's headline capabilities — the primitives that decide what a server and client actually do together. This lecture covers three smaller, supporting mechanisms that keep a real, long-running connection honest: **notifications**, **roots**, and **progress reporting**. None of the three is a primitive in the same sense as a tool or a resource, but the exam treats all three as fair game, because a protocol meant to support real, stateful, long-running sessions needs exactly this kind of connective tissue.

## Notifications: the server telling the client something changed

A **notification** is a message a server sends to a client, outside the normal request-response flow, to signal that something relevant has changed since the client last checked. The clearest example is a change in the server's own capabilities: if a server's available tools, resources, or prompts change during a session — a new tool becomes available, a resource is deleted, a resource's content is updated — the server can send a notification telling the client to re-run discovery rather than continuing to operate on a now-stale list it fetched once at connection time.

This matters because MCP connections are often long-lived, and a server's capability surface isn't necessarily static for the life of that connection. A filesystem server might notify a client that a watched file's content changed, so that a client holding that file as an attached resource knows to re-fetch it rather than silently working from stale content. A server managing some external system's configuration might notify the client that a tool has become temporarily unavailable — a maintenance window, a permissions change — rather than letting the client discover that the hard way, via a failed call. Notifications are what let MCP's discovery model be dynamic rather than a one-time snapshot taken at connection time and trusted uncritically for the rest of the session.

## Roots: the client telling the server its boundaries

A **root** is, in effect, the opposite direction of communication: the client telling the server what it's actually allowed to operate within — typically expressed as a filesystem directory or a set of directories the server should treat as its operating boundary. A client connecting a filesystem-oriented MCP server, for instance, can declare one or more roots — "you may operate within this project directory and nowhere else" — so the server has an explicit, protocol-level signal about scope rather than needing to infer, or worse, simply trust, what it should and shouldn't touch on the host machine.

Roots are a direct, practical extension of the least-privilege principle the Tool Layer's governance lecture established for credential scoping, applied specifically to the filesystem-boundary case: rather than relying entirely on a server's own internal logic (or the model's judgment) to stay within an appropriate scope, the client can communicate that scope explicitly at the protocol level, giving the server a legitimate, structural signal about its boundaries rather than leaving "don't touch files outside the project" as an unenforced convention. A well-behaved server respects the roots it's given; a client with real security requirements shouldn't rely on that alone, but roots meaningfully reduce the chance of an in-scope, well-intentioned server accidentally overreaching due to ambiguity about what it was meant to operate on.

## Progress reporting: keeping long operations legible

Some tool calls made through an MCP server take real time — processing a large file, running a long query, executing a multi-step external workflow — and a client (and, by extension, a user) waiting on a single opaque request with no feedback for an extended period is a poor experience and a genuinely ambiguous state to be in: is the operation still running, or has it silently stalled? **Progress reporting** is the protocol mechanism that addresses this: a server executing a long-running operation can send incremental progress updates back to the client over the course of that operation, rather than the client only ever hearing back once, at completion, with no visibility in between.

This connects directly to the Tool Layer's discussion of designing tools around real operational needs rather than an idealized instant-response case: a tool whose underlying operation can genuinely take minutes needs a way to communicate "still working, X% done" or "still working, currently on step 3 of 7," and progress reporting is the standard, protocol-level way an MCP server does that rather than every server inventing its own bespoke polling or streaming convention.

## A worked example: all three in one session

Picture a client with a filesystem MCP server connected, mid-session on a long refactoring task. At connection time, the client declares a root of `/home/user/project`, telling the server upfront that its operations should stay scoped to that directory regardless of what the underlying host filesystem otherwise contains. Partway through the session, the model invokes a tool that reformats every file in a large subdirectory — a genuinely slow operation — and the server sends a stream of progress notifications ("processed 40 of 210 files") so the client can show the user this is actively working rather than stalled. Near the end of the session, a file the server was watching changes outside of any tool call the model made — perhaps another process modified it — and the server sends a notification that its resource listing has changed, prompting the client to re-fetch before trusting that resource's content again. Three different mechanisms, three different moments, three different directions of communication, all in service of the same underlying goal: keeping a long, stateful session accurate about scope, progress, and freshness without requiring the model itself to reason about any of that bookkeeping.

## Why these three belong together

Notifications, roots, and progress reporting share a common role even though they point in different directions: each is infrastructure for keeping a stateful, potentially long-running connection legible and well-scoped, rather than infrastructure for accomplishing a task directly the way a tool or resource does. None of the three is something a model decides to invoke — they're mechanisms the protocol itself uses to keep client and server honestly synchronized about state, scope, and progress over the life of a connection that may run far longer than a single request-response exchange.

**Why it matters for the exam:** Expect each of the three to be tested with its direction and purpose distinct from the others — notifications flow server-to-client and signal a capability or content change; roots flow client-to-server and declare an operating boundary; progress reporting flows server-to-client during a long-running call and communicates incremental status. A common distractor pattern is swapping the direction (attributing a root to the server or a notification to the client) or conflating progress reporting with a notification generally — keep the three distinct by role, not just by name.
