---
title: Context in Large Codebases
parent: The Context Layer
nav_order: 8
duration: "23:55"
---

# Context in Large Codebases
{: .no_toc }

*Video duration: 23:55*

Every technique covered so far in this module assumes a workable premise: that the material relevant to a task, once identified, is small enough to fit in a context window after reasonable summarization and pruning. Large codebases break that premise outright. A mature production repository can run to millions of lines across thousands of files — no amount of compaction makes that fit in any context window, and no reasonable person would want it to even if it could, because the overwhelming majority of it is irrelevant to any single task. Context management in a large codebase isn't a scaled-up version of the techniques covered earlier; it's a different problem with a different center of gravity, and this lecture is about what changes.

## The shift from "what to keep" to "what to find"

In a small project, the dominant context management question is subtractive: given everything that's already naturally in scope — a handful of files, a compact `CLAUDE.md`, a short history — what should be trimmed, summarized, or dropped to stay lean? In a large codebase, the dominant question is the opposite. It's additive and selective: out of a corpus that can never be fully in scope, what narrow slice should be pulled *into* context in the first place? The center of gravity moves from compaction to **retrieval** — the process of searching, narrowing, and selecting the specific material a task actually needs, out of a much larger universe that's deliberately left untouched.

This reframing matters because it changes where the architectural effort goes. Managing context in a small project is largely about discipline over what's already there. Managing context in a large codebase is largely about the quality of the *search and selection* step that decides what gets there at all — get that step wrong, and no amount of downstream summarization skill will save the task, because either the wrong material was pulled in (diluting the window with irrelevant code, the same signal-dilution problem from earlier in this module, just caused upstream) or the right material was never pulled in at all, which is functionally identical to the missing-information ambiguity problem covered two lectures back.

## Retrieval and search-driven narrowing

The mechanics of narrowing a large codebase down to a workable context generally combine a few complementary techniques, and a well-built agentic coding system typically uses more than one of them together rather than relying on a single approach.

**Keyword and pattern search** is the most direct technique: grep-style search for a specific symbol, string, or pattern across the corpus. It's fast, precise when you already know roughly what you're looking for (a function name, an error message, a configuration key), and cheap in terms of context cost, because it returns matching locations rather than requiring the whole corpus to be scanned by the model itself. Its limitation is that it only finds what it's told to look for in the terms it's told to look in — a search for a function's name won't surface a caller that references it indirectly through an alias or a dynamically constructed call, and it won't help at all when the task is conceptual rather than lexical ("where does this system handle retries" when no code literally contains the word "retry").

**Semantic or embedding-based retrieval** addresses that gap by searching for conceptual relevance rather than literal text match, using vector representations of code or documentation to find material that's related in meaning even without shared vocabulary. This is closer to what a general RAG pipeline does over a document store, applied to a codebase instead — and it's valuable precisely where keyword search falls short, at the cost of being fuzzier and occasionally surfacing material that's related in theme but not actually relevant to the specific task at hand.

**Structural and symbol-level navigation** uses the codebase's own structure — its dependency graph, its call graph, its import relationships — to move outward from a known starting point along real code relationships rather than text similarity. Given one file or function known to be relevant, structural navigation can answer "what calls this," "what does this depend on," and "what else lives in this module" far more reliably than either keyword or semantic search, because it's following the actual architecture rather than approximating relevance from text. This is often the technique that turns an initial keyword or semantic hit into a properly scoped, complete-enough context — the first search finds *a* relevant location, and structural navigation from there finds the rest of what's actually connected to it.

| Technique | Best for | Limitation |
|---|---|---|
| Keyword/pattern search | A known symbol, string, or exact term | Misses conceptual matches and indirect references |
| Semantic/embedding retrieval | Conceptual relevance without shared vocabulary | Fuzzier; can surface thematically related but irrelevant material |
| Structural/symbol navigation | Following call graphs, dependencies, and imports outward from a known point | Needs an existing starting point to navigate from |

**Iterative, agent-driven exploration** ties these together in practice: rather than trying to determine the full relevant scope in one shot, an agent working a large-codebase task typically explores incrementally — an initial search narrows to a plausible area, reading that area's content reveals what else needs to be pulled in, another targeted search or navigation step follows, and the process repeats until the agent has assembled just enough context to act, and no more. This is a meaningfully different pattern from a single retrieval step feeding directly into a single generation step, and it's part of why agentic coding tools are structured as an iterative loop with search and file-reading tools available throughout, rather than a one-shot "retrieve everything relevant, then answer."

## File-level versus symbol-level context selection

A related and consequential design decision is the granularity at which retrieved material actually enters context: whole files, or specific symbols (functions, classes, methods) extracted from within them.

Pulling in whole files is simpler and preserves surrounding context that might matter — the imports at the top, adjacent functions that clarify a convention, comments that explain intent for the file as a whole. It's the right default when a file is reasonably sized and reasonably cohesive, because the overhead of the "extra" material is usually worth the safety of not silently missing something adjacent that mattered.

Pulling in specific symbols is more precise and dramatically cheaper in token cost when a file is large but the actual relevant material within it is small — a thousand-line file where only one thirty-line function is relevant to the task at hand. The risk with symbol-level extraction is the mirror image of the whole-file risk: it can strip away exactly the surrounding context that would have clarified how that symbol is meant to be used, what invariants the surrounding code assumes, or what a sibling function does slightly differently for a reason that only becomes clear by seeing it alongside the one that was extracted.

There's no universal right choice between the two; it's a judgment call that should track the same size-versus-cohesion logic that governs summarization elsewhere in this module. A tightly-scoped utility file where every function is self-contained is safe to extract from at the symbol level. A file whose functions closely interrelate — sharing module-level state, calling each other, implementing different branches of one conceptual operation — is much more dangerous to fragment, because the file's coherence *is* part of its meaning, and symbol-level extraction can silently discard that.

## How this differs from small-project, CLAUDE.md-style context

It's worth being explicit about the contrast with the small-project pattern, because the exam likes to test whether you understand that these aren't just different scales of the same technique — they're different techniques suited to different situations, and applying the wrong one is a real, recognizable mistake.

A `CLAUDE.md`-centered approach works well when the relevant context is relatively stable, relatively small, and genuinely applies broadly across most tasks in the project — conventions, architectural notes, and constraints that are true regardless of which specific file a given task touches. Loading that content automatically, on every turn, is efficient precisely because it's not being selectively retrieved for a specific task; it's baseline context that's cheap enough to always include and valuable enough to be worth the standing cost.

That approach breaks down at scale for a straightforward reason: a corpus large enough that no meaningful fraction of it is broadly relevant to most tasks can't be served by a static, always-loaded file, because there's no small subset of "the important stuff" that's actually important across the board — what's relevant is determined by the specific task, not by the project as a whole. Trying to force a `CLAUDE.md`-style pattern onto a large, heterogeneous codebase either produces a `CLAUDE.md` so large it becomes its own context management problem (paying its cost on every single turn regardless of relevance, the exact standing-tax issue flagged earlier in this module), or one so thin it captures only the most universal conventions and provides no real help finding anything task-specific.

The corollary is equally important: retrieval-driven, search-based context selection is generally *overkill* for a small, stable project, where the relevant context really can be enumerated once and loaded automatically without much waste. Building an elaborate semantic search and symbol-extraction pipeline for a project small enough to fit comfortably in a `CLAUDE.md` and a handful of always-loaded files adds real engineering complexity and latency for a problem that a much simpler, static approach already solves well. Matching the context strategy to the actual scale and heterogeneity of the corpus — not defaulting to the more sophisticated approach because it seems more rigorous — is itself part of the judgment this domain is testing.

{: .important }
> **Match context strategy to the corpus's actual scale and heterogeneity — never default to whichever technique seems more sophisticated.** A static `CLAUDE.md`-style approach forced onto a huge, heterogeneous codebase fails just as reliably as an elaborate retrieval pipeline is overkill for a small, stable project.

## Managing retrieved context once it's in the window

Getting the right material into context is only half the problem; the other half is managing it once it's there, and this is where large-codebase context work reconnects with the general techniques from earlier in this module. Retrieved code and documentation still competes for space the same way any other context does, and it still needs the same discipline: don't let stale retrieval results linger past the point they're useful, summarize or discard exploratory dead ends the same way a debugging session's failed attempts get compacted, and be conscious that each additional file pulled in during iterative exploration is **a real cost against everything else in the window**, not a free addition just because it came from a search tool rather than the conversation itself.

This is also where the noise-contamination concern from the isolation discussion becomes directly relevant. An agent's iterative exploration of a large codebase — trying one search, following a dead-end lead, backtracking, trying another — generates exactly the kind of intermediate noise that shouldn't necessarily flow forward into whatever consumes the agent's final output. A subagent tasked with locating and explaining how a particular feature is implemented should generally return a clean, purpose-built summary of what it found and where, not the full, messy transcript of every search query and false lead it tried along the way. The exploration process is valuable while it's happening; most of it stops being valuable the moment the actual answer is found.

## Staleness and drift in a large, actively changing codebase

One more consideration is specific to large codebases and worth naming explicitly: unlike a static document corpus, an actively developed codebase changes underneath any retrieval system, sometimes rapidly. Retrieved context — whether from keyword search, semantic search, or a cached index — always carries some risk of describing a version of the code that's already been superseded by a commit made after the index was last updated. This is a provenance concern in miniature (the topic gets its full treatment two lectures ahead), but it's worth flagging here because it's a failure mode specific to this scale: a small, hand-maintained `CLAUDE.md` is updated deliberately and rarely goes stale without someone noticing; an automated retrieval index over a fast-moving codebase can drift silently, and an agent has no way to know it's working from outdated material unless the retrieval system itself surfaces some signal of currency — an index timestamp, a "last synced" marker, or a live re-check against the actual current file rather than a cached representation of it.

## A worked example: tracing a bug through an unfamiliar monorepo

It's worth walking through how these pieces actually combine in a realistic session, because the value of combining techniques is easier to see than to state abstractly. Suppose an agent is asked to fix a bug: a notification is occasionally sent twice to the same user. The repository is a large monorepo the agent has no prior familiarity with, and nothing about "notification," "duplicate," or "twice" points to an obvious single file.

The agent starts with a keyword search for terms likely to appear near the relevant code — "notification," "send," "dedupe," "idempotent" — and gets back a scattered list of dozens of hits across the notification service, several tests, and a handful of unrelated files that happen to share vocabulary (a "send" function in an unrelated email module, for instance). None of this alone is enough context to act on, and pulling all of it into the window at once would recreate exactly the signal-dilution problem this module keeps returning to.

Instead, the agent reads just enough of the two or three most plausible hits to identify the actual notification-dispatch function, then switches techniques: structural navigation from that function outward, following its callers, to see every place in the codebase that can trigger a send. This surfaces something keyword search alone wouldn't have made obvious — the function is called from two different code paths, a scheduled retry job and a real-time event handler, and nothing in either path checks whether the other has already sent the same notification. That's the actual bug, and it only became visible once search-driven narrowing (which found the function) was followed by structural navigation (which found every way into it).

Notice what didn't happen: the agent never loaded the notification service's full source tree, never read the unrelated email module's "send" function beyond confirming it was irrelevant, and never kept the initial keyword search's full result list in context once it had served its purpose of pointing toward the real function. Each step's output fed the next step's narrower search, and material that had done its job was left behind rather than accumulated. That discipline — treat each retrieval step's result as a stepping stone to the next query, not as permanent context — is what keeps an iterative, agent-driven exploration from simply reproducing the "just include everything" failure mode one search result at a time.

## Index freshness as its own context decision

One more practical wrinkle deserves a name: many retrieval systems over large codebases work against a pre-built index — an embedding index for semantic search, a symbol table for structural navigation — rather than scanning the live file system on every query, because scanning everything on every query doesn't scale any better than loading everything into context would. That's a reasonable engineering trade-off, but it reintroduces the staleness concern from earlier in this lecture at the infrastructure level: the index itself is a piece of context input with its own currency, and if it isn't rebuilt often enough relative to how fast the codebase actually changes, every retrieval built on top of it inherits a hidden risk of describing code that no longer exists in that form. Deciding how the index is kept current — on every commit, on a schedule, or on demand before a session that needs precision — is itself a context management decision, not just an infrastructure detail to delegate away without a second thought.

**Why it matters for the exam:** Large-codebase scenarios are a reliable exam pattern, usually framed as a choice between a static, `CLAUDE.md`-style approach and a retrieval-driven approach for a described project. The correct answer tracks the actual scale and heterogeneity of the corpus described in the scenario, not a general preference for one technique over the other — and a well-constructed question will sometimes describe a small project and expect you to recognize that the elaborate retrieval pipeline is the wrong, over-engineered answer, exactly as often as it describes a large one and expects the reverse.

<!-- prevnext:start -->

---

| [&larr; Previous: Should I Stay? Or Should I Propagate?](07-should-i-stay-or-should-i-propagate) | [Next: An Emergent, Hyper-Specialized Local Use Case &rarr;](09-an-emergent-hyper-specialized-local-use-case) |
|:---|---:|

<!-- prevnext:end -->
