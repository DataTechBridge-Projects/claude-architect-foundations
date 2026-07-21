---
title: Error Response Design
parent: The Tool Layer
nav_order: 4
duration: "17:20"
---

# Error Response Design
{: .no_toc }

*Video duration: 17:20*

A tool that always succeeds doesn't need careful error design, and a tool that never fails doesn't exist. Every real tool — a file read, an API call, a database query, a shell command — fails sometimes, for reasons ranging from trivial (a typo in a filename) to systemic (a downstream service is down). The question this lecture answers is not whether tools fail, but what happens in the model's reasoning the instant after they do, and how the shape of the error you return determines whether that instant produces a sensible recovery or a cascading mess.

## The error message is a message to a reasoning system

The habit to unlearn, if you're coming from traditional software engineering, is writing error messages the way you'd write them for a human developer debugging in a terminal. A stack trace, an HTTP status code with no accompanying explanation, a bare exception class name — these are all completely reasonable outputs when the consumer is a human who can read source code, check logs, and apply general engineering intuition to figure out what actually went wrong and what to do about it. None of that context is available to the model receiving a tool result. The model sees exactly the text you return as the tool result and nothing else about the failure's underlying cause, unless you put that information in the text.

This means an error response isn't a debugging artifact — it's an instruction to a reasoning system about what happened and, ideally, what a sensible next move looks like. Every error you design should be evaluated by one question: *if this were the only information available to decide what to do next, would that decision be a good one?* A `FileNotFoundError: [Errno 2] No such file or directory: 'config.yml'` traceback, dumped verbatim as a tool result, technically contains the information a model needs (the file doesn't exist), but it's buried in formatting optimized for a different reader, and it tells the model nothing about what to try instead — was the path wrong, is the file expected to exist yet, should it search for the file elsewhere? A well-designed error answers those questions explicitly rather than making the model infer them from a traceback's shape.

## Anatomy of a good tool error

A tool error response that actually helps a reasoning model recover tends to have three components, and it's worth treating each as a required field rather than an optional nicety.

**What went wrong, specifically.** Not "the operation failed," but the precise failure: which field was invalid, which resource wasn't found, which constraint was violated. Specificity is what turns an error from a dead end into a data point the model can reason from. "Invalid input" gives the model nothing to work with; "the `status` field must be one of 'open', 'closed', or 'all' — received 'active'" tells the model exactly what to fix and, because it restates the valid values, often lets the model correct the call on the very next attempt without a second failed round-trip.

**Why, in terms that map to something the model can act on.** Root cause matters less than actionable cause. A downstream timeout, from the model's point of view, doesn't need an explanation of TCP retransmission — it needs to know whether this is the kind of failure that might succeed if tried again, or whether it reflects some enduring condition (the resource genuinely doesn't exist, the request is malformed, permission is denied) that no number of retries will fix. This is the retryable-versus-non-retryable distinction, and it's the single most consequential piece of information an error can carry, covered in depth below.

**What a valid alternative would look like, when one exists.** The strongest error responses don't just describe the failure, they point toward the fix. If a lookup fails because an ID doesn't exist, and the tool (or a sibling tool) can offer close matches or a search alternative, surfacing that in the error — "no issue found with ID 4821; did you mean 4812? Alternatively, use search_issues to find it by keyword" — turns a dead-end failure into a one-step recovery. This is genuinely more design work than just returning "not found," but it's where the return on investment in error design is highest, because it collapses what would otherwise be a multi-turn recovery (fail, reason about alternatives, try something, maybe fail again) into a single corrected retry.

## Retryable versus non-retryable: the load-bearing distinction

If you take only one idea from this lecture, take this one: every tool error should make it clear, explicitly or through consistent convention, whether the failure is something a retry could plausibly fix or something no amount of retrying will fix. Collapsing this distinction — returning the same generic "the operation failed" for both a transient network timeout and a permanently invalid request — is one of the most common and most damaging tool-design mistakes, because it leaves the model with genuinely insufficient information to choose a sensible next action, and the two sensible next actions are opposites.

**Retryable failures** are transient by nature: a timeout, a rate limit, a momentary service unavailability, a lock contention error. The correct model behavior, given a clearly retryable error, is usually to retry — possibly with a brief pause, possibly with the same arguments unchanged, since the arguments weren't the problem. A well-designed error response for this category should say so plainly: "Request timed out after 30s. This is likely transient — retrying the same request may succeed." Some tool implementations go further and return a `retryable: true` field or a suggested backoff interval alongside the message, which is worth doing when the calling harness (rather than the model itself) is the one responsible for the actual retry loop — see the next lecture for how that division of labor typically works.

**Non-retryable failures** are structural: a malformed request, a resource that genuinely doesn't exist, a permission the caller doesn't have, a business-rule violation (attempting to refund an order that was never charged). No amount of identical retrying fixes any of these — the model needs to change something about its approach: different arguments, a different tool, or an escalation to a human. An error response for this category should say so just as plainly, and ideally point toward what needs to change rather than just refusing: "Permission denied: this API key does not have write access to the `production` environment. Read access is available; write actions require an elevated credential this session does not have." That response doesn't just say no — it tells the model *why* retrying with the same credential will never work, which is exactly the information that prevents a model from burning several turns hammering the same doomed call.

| | Retryable | Non-retryable |
|---|---|---|
| Nature of the failure | Transient — timeout, rate limit, momentary lock | Structural — malformed request, missing resource, denied permission, business-rule violation |
| Correct model action | Retry, possibly after a brief pause, same arguments | Change something: different arguments, a different tool, or escalate |
| What the error should say | "This is likely transient — retrying may succeed" | "Retrying will not work — here's why, and what to try instead" |
| Example | "Request timed out after 30s. Retrying the same request may succeed." | "Permission denied: this API key lacks write access to production." |

Making this distinction legible doesn't require a rigid taxonomy in every tool's schema, though some systems do formalize it with an explicit `error_type` or `retryable` field in the tool result, which is a reasonable pattern when a tool set is large enough that consistent machine-parseable conventions pay for themselves. At minimum, the language of the message itself should make the distinction unambiguous, since that's the layer the model is actually reading, with or without a formal field alongside it.

## Avoiding raw stack traces and internal implementation detail

It's worth being explicit about what *not* to return, because the path of least resistance in an implementation is often to catch an exception and pass its string representation straight through as the tool result. Resist this. A raw stack trace or an internal exception message has several problems as a tool error specifically:

- **It's optimized for the wrong reader.** A stack trace's structure — file paths, line numbers, call frames — is designed to help a human developer locate a bug in source code. A model reasoning about what to do next doesn't benefit from a call stack; it benefits from a plain statement of what failed and why.
- **It leaks implementation detail that constrains you later.** If a tool's error responses are effectively "whatever the underlying library happens to raise," then refactoring that underlying implementation — swapping a database driver, changing an internal library — silently changes the error strings the model has been conditioned by, in context, to interpret a certain way. Designed error messages are a stable interface in the same way a well-designed API's success responses are; raw exception passthrough is not.
- **It can be actively confusing or misleading.** A database-level constraint violation error, passed through raw, might reference table and column names that mean nothing in the tool's own vocabulary — the model has to reverse-engineer what "duplicate key value violates unique constraint 'issues_external_id_key'" means for the `create_issue` tool it just called, when a designed error would simply say "an issue with this external ID already exists — use update_issue instead, or omit external_id to create a new one."
- **It can leak information that shouldn't be exposed at all.** Internal file paths, hostnames, database schema details, or fragments of internal system architecture showing up in raw error text is a minor information-disclosure risk in any system, and becomes a slightly larger one in an agentic system where that text might get echoed back to an end user in the model's eventual response, or logged somewhere less carefully controlled than your application's own error-handling code.

The fix in all these cases is the same: catch the underlying failure at the tool boundary and translate it deliberately into a designed error message, using the anatomy above — what went wrong, whether it's retryable, and what a valid alternative looks like — rather than letting whatever the implementation happened to throw leak through unfiltered.

## A worked comparison

Consider a `create_calendar_event` tool that fails because the requested time slot conflicts with an existing event. A raw-passthrough error might read: `ConflictError: 409 - resource conflict at /calendar/v3/events, event_id=None`. A designed error, applying everything above, might instead read: `Cannot create event: the requested slot (2pm–3pm, July 22) conflicts with an existing event, "Budget Review" (1:30pm–3pm). This is not retryable with the same time — choose a different slot, or use list_events to see what's already scheduled that afternoon.`

The second version costs more to write — it required the tool's implementation to actually look up and name the conflicting event, not just report that a conflict occurred — but it converts what would otherwise be at least a two-turn recovery (fail, ask the model to reason about why, maybe call `list_events` speculatively, then retry) into a response the model can act on immediately, in many cases correctly on the very next tool call. That compounding effect — fewer wasted turns, fewer redundant clarifying tool calls, a shorter overall path to task completion — is the concrete, measurable payoff of investing in error design, and it's the reason experienced tool designers treat the failure path with at least as much care as the success path, not as an afterthought bolted on once the happy path works.

## Errors as part of the tool's interface, not exceptions to it

The overarching mental shift this lecture asks for is to stop treating error responses as an edge case handled separately from "real" tool design, and start treating them as a first-class part of the tool's interface — as deliberately designed as the success schema, and reviewed with the same rigor. A tool definition that specifies a rich, well-typed success response but leaves failure as an untyped, unexamined string is only half-designed, and in practice, for any tool that gets used enough, the failure path is exercised nearly as often as the success path. Given how much of an agent's actual behavior is shaped by what it does immediately after a failure, that's not a minor gap — it's frequently the difference between a system that degrades gracefully under real-world conditions and one that doesn't.

**Why it matters for the exam:** Expect scenario questions that present a tool's error response verbatim and ask you to identify what's missing — usually the retryable/non-retryable signal, or a raw stack trace standing in for a designed message — and to describe what a better response would say instead. You should also be comfortable explaining *why* a given error design leads to a specific, predictable model behavior (blind retry, premature give-up, hallucinated workaround), since that causal link between error design and recovery behavior is the thread connecting this lecture to the next one.

<!-- prevnext:start -->

---

| [&larr; Previous: Structure with Prose as the Fallback](03-structure-with-prose-as-the-fallback) | [Next: Multiple Pathways &rarr;](05-multiple-pathways) |
|:---|---:|

<!-- prevnext:end -->
