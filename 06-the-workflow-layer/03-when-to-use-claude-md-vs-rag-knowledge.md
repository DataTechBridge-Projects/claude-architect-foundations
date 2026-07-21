---
title: When to use CLAUDE.md vs. RAG Knowledge?
parent: The Workflow Layer
nav_order: 3
duration: "0:46"
---

# When to use CLAUDE.md vs. RAG Knowledge?
{: .no_toc }

*Video duration: 0:46*

`CLAUDE.md` and retrieval (RAG) solve the same underlying problem — getting the right knowledge in front of the model — for opposite kinds of knowledge. `CLAUDE.md` is right for information that's small, stable, and *always* relevant: build commands, architectural conventions, hard constraints. It's loaded on every turn, so its cost is paid every session whether or not that session needs it. RAG is right for knowledge that's large, changes frequently, or is only relevant to specific queries — a full API reference, historical tickets, an evolving knowledge base. Retrieving it on demand avoids paying context cost for knowledge most sessions never touch.

| | `CLAUDE.md` | RAG / retrieval |
|---|---|---|
| Size | Small | Large |
| Volatility | Stable | Frequently changing |
| Relevance | Relevant to nearly every session | Relevant only to specific queries |
| Cost pattern | Paid every session, needed or not | Paid only when retrieved |

A concrete example makes the split obvious. Say a project exposes forty REST endpoints. The fact that "all routes live under `src/routes`, one file per resource, and every handler must call the auth middleware first" is exactly the kind of thing every session benefits from — it's a few lines, it never changes week to week, and it shapes how Claude should write *any* new route. That belongs in `CLAUDE.md`. The full request/response schema for all forty endpoints, by contrast, is hundreds of lines that any given session will mostly never touch — a task that only concerns the billing endpoint has no use for the shipping and inventory schemas sitting in context the whole time. Indexing that reference and retrieving only the two or three endpoints relevant to the current task is what RAG is for: the same knowledge, made available on demand instead of paid for unconditionally.

The two axes worth separating are **size** and **volatility**, and they don't always move together. Something can be small but volatile — this week's on-call rotation, today's known flaky test — where neither `CLAUDE.md` nor RAG is really the right tool, because both assume the underlying content is worth indexing or hard-coding rather than looked up live. Something can also be large but genuinely stable, like a finalized architecture decision record, where the "large" part still argues for retrieval even though "stable" might tempt you toward `CLAUDE.md`. The deciding question is less "is this static" and more "does *every* session need it, or only the sessions that happen to touch this specific slice."

This connects directly to the `CLAUDE.md` anti-patterns covered in the previous lecture: treating `CLAUDE.md` as a dumping ground is precisely the mistake of using it for knowledge that RAG should be handling instead. It's also worth distinguishing this from Skills, covered next — Skills are the on-demand mechanism for *procedural* knowledge (how to do something), loaded into the same agent's context when a task matches. RAG is the on-demand mechanism for *declarative* knowledge (what is true, or what the reference says) — different shape of content, same underlying "don't pay for it until you need it" principle.

<!-- prevnext:start -->

---

| [&larr; Previous: Configuring CLAUDE.md](02-configuring-claude-md) | [Next: Skill Configuration &rarr;](04-skill-configuration) |
|:---|---:|

<!-- prevnext:end -->
