---
title: Prompting Best Practices
parent: The Prompt Layer
nav_order: 2
duration: "17:35"
---

# Prompting Best Practices
{: .no_toc }

*Read duration: 17:35*

If there's one habit that separates prompts written by architects from prompts written by hobbyists, it's this: architects assume the model will do exactly what the prompt says, not what the author meant. That sounds obvious once stated, but it's violated constantly, because human communication relies on shared context, tone, and inference that a language model doesn't automatically have access to unless it's explicitly given. Prompting best practices, at the structural level, are really a set of techniques for closing that gap — for making the prompt's literal content match the author's actual intent so closely that there's no room left for the model to guess wrong.

## Clear Task Framing

Every prompt is implicitly answering the question "what am I being asked to do, and why?" A well-framed prompt answers it explicitly. This means stating the task itself, but it also means giving the model enough surrounding context to understand *why* the task matters, because that context shapes judgment calls the instructions can't fully anticipate.

Consider the difference between two prompts asking for the same underlying work:

- "Summarize this document."
- "Summarize this document for a busy VP of Sales who has 90 seconds before a board meeting and needs to know only what changed since last quarter and what decision, if any, they're being asked to make."

Both prompts produce "a summary." They will not produce the same summary. The first leaves length, tone, and emphasis entirely to the model's default assumptions, which are generic because the prompt gave it nothing to specialize against. The second gives the model an audience, a time constraint, and a specific informational goal — and every one of those details will visibly shape the output, from sentence length to what gets left out. This is the essence of clear task framing: don't just name the task, situate it. What is this output *for*, who consumes it, and what does success look like from their perspective?

Task framing also means being explicit about scope. "Review this pull request" is an invitation for the model to decide, on its own, whether "review" means checking for bugs, checking for style violations, checking for security issues, checking for test coverage, or all four. If you actually want all four, say so — and if you want only one, say that instead. Every dimension of scope you leave unstated is a dimension the model has to infer, and inference is where reliability erodes, especially across the range of variation a production system will actually encounter.

## Explicit Success Criteria

A task description tells the model what to do. Success criteria tell it what "done well" looks like — and without them, the model has no way to self-correct or prioritize when trade-offs arise, which they always do in anything beyond trivial tasks.

Good success criteria are specific and, where possible, checkable. Compare "write good tests for this function" against "write tests that cover the happy path, at least two edge cases involving empty or null input, and one case that exercises the documented error condition; each test should be independently runnable and should not depend on execution order." The second version gives the model a checklist it can hold its own output against before returning it. This matters more than it might seem, because models — like people — tend to produce better work when they know how they'll be judged. A prompt that states success criteria up front is, in effect, giving the model a rubric to self-evaluate against mid-generation, which is cheaper and often more effective than a separate review pass added after the fact (a technique this module returns to directly in the lecture on quality and feedback enforcement).

Success criteria are also where you encode the constraints that matter operationally but aren't part of the task's core logic: length limits, tone requirements, formatting rules, things the output must never contain (secrets, internal file paths, competitor names), and things it must always include (a disclaimer, a specific field, a citation). If a constraint genuinely matters to whether the output is usable, it belongs in the prompt as an explicit, checkable criterion — not as an assumption you're hoping the model shares.

## System Content vs. User Content

Claude's API distinguishes between a **system prompt** — persistent instructions that establish the model's role, constraints, and behavior for the entire interaction — and **user content**, the specific request or data for a given turn. Understanding this distinction, and using it correctly, is one of the more consequential structural choices in prompt architecture.

The system prompt is the right place for anything that should hold true across every request in a session or every invocation of an agent: the model's persona and role ("You are a code review assistant for a Python monorepo"), durable constraints ("Never suggest disabling a security check; flag it for human review instead"), output format requirements that apply universally, and the general behavioral posture you want (cautious and conservative vs. fast and exploratory, for instance). Because it's set once and applies throughout, the system prompt is also the natural home for anything expensive to restate — long-form instructions, few-shot examples, or reference material that would be wasteful to repeat on every turn.

User content is the right place for what's actually changing: the specific document to summarize, the specific ticket to triage, the specific code diff to review. Mixing these up has real costs. If task-specific data gets pushed into the system prompt, you lose the ability to cache and reuse that system prompt efficiently across many different requests, and you make it harder to reason about what's fixed versus what's varying in your architecture. If durable behavioral rules get pushed into user content instead, they become easy to omit accidentally on some call sites and not others, producing exactly the kind of inconsistency this whole module is trying to engineer away. A useful mental model: the system prompt is the model's job description; user content is today's specific assignment. Keep the job description stable and put it in one place; let the assignment vary.

This distinction also matters for security posture, which the later lecture on sanitization revisits: content that comes from untrusted sources (a user's raw input, a scraped web page, a document a customer uploaded) should generally live in clearly demarcated user content, not be woven into system-level instructions, precisely because system-level instructions are the ones the model is trained to trust most.

| Belongs in the system prompt | Belongs in user content |
|---|---|
| Persona and role ("You are a code review assistant...") | The specific document, ticket, or diff for this call |
| Durable constraints that hold across every call | Data that changes call to call |
| Universal output format requirements | Untrusted external input (raw user text, scraped pages) |
| Few-shot examples and reference material expensive to restate | Anything that shouldn't be baked into a cached, reused prompt |

{: .important }
> Never let untrusted content — raw user input, scraped text, an uploaded document — get woven into system-level instructions. System prompts are what the model is trained to trust most, so blending untrusted text into them is a direct path to prompt injection succeeding.

## Avoiding Ambiguity

**Ambiguity is the single largest source of unreliable prompt behavior**, and it's insidious because it's invisible to the person who wrote the prompt — you know what you meant, so the sentence reads as clear to you even when it supports multiple reasonable readings. A few concrete patterns are worth naming because they show up constantly:

**Underspecified pronouns and references.** "Check if it matches and update the record" — what is "it," and which record? In a long prompt with several entities in play, referential ambiguity compounds. Name things explicitly rather than relying on the model (or a human reader) to track antecedents across paragraphs.

**Overloaded terms.** Words like "review," "validate," "process," or "check" carry different meanings depending on domain, and a model will pick some meaning rather than stopping to ask which one you intended — unless you've explicitly told it that asking is an option, and even then it will often not ask about a term it has no specific reason to think is contested. If "validate this input" could mean "check the schema," "check business rules," or both, say which.

**Implicit conditionals.** "If the request looks urgent, escalate it" sounds like an instruction, but "looks urgent" is doing enormous invisible work. What signals urgency — specific keywords, a stated deadline, sentiment? Either define the trigger condition concretely or accept that the model's judgment will vary run to run, which may be fine for some tasks and unacceptable for others. Know which case you're in before you ship the prompt.

**Competing instructions.** Longer prompts sometimes accumulate rules added at different times that quietly conflict — "always respond in under 200 words" alongside "always include a full citation list" for a task where citations alone can exceed 200 words. When instructions compete, the model has to choose which one to honor, and it may not choose the way you'd want. Periodically re-reading a prompt end to end, specifically hunting for contradictions, is a cheap habit that catches this class of bug before it reaches production.

## Being Explicit Rather Than Relying on Inference

Underlying all of the above is a single unifying principle: **explicit beats implicit**, every time reliability matters. Claude models are very good at inference — filling in gaps, reading between the lines, extrapolating from partial instructions — and that capability is genuinely useful for open-ended, exploratory tasks where you want the model's judgment to do real work. But it is exactly the wrong capability to lean on for anything that needs to behave the same way across hundreds or thousands of invocations, because inference is where variance lives. Two runs of the same slightly-ambiguous prompt can land on two different, individually reasonable interpretations, and in a production system that variance shows up as a bug report, not as a delightful surprise.

The practical discipline this implies is: whenever you notice yourself thinking "the model will probably figure out what I mean," treat that as a signal to go back and say what you mean directly. This applies to formatting ("respond in markdown with these three headers, in this order" beats "format it nicely"), to edge cases ("if the input is empty, return an empty list rather than an error" beats leaving empty input unaddressed), and to priority ("if these two goals conflict, prioritize correctness over brevity" beats hoping the model resolves the tension the way you would have). None of this means padding a prompt with redundant restatements — verbosity for its own sake doesn't improve reliability and can dilute the instructions that actually matter. It means every instruction that needs to hold should be stated in a form specific enough that a reasonable reader — human or model — has only one way to interpret it.

## A Worked Example

It helps to see these principles applied together against a single prompt, revised in stages. Suppose the starting point is: "Review this pull request and leave comments." That single sentence violates nearly every principle above at once. Task framing is missing — review for what, exactly, and for whose benefit? Success criteria are absent — what does a good review accomplish that a bad one doesn't? There's no distinction drawn between what belongs in a persistent system prompt (the reviewer's general standards, which should hold across every PR this system ever reviews) versus what's specific to this call (the actual diff). And "review" itself is exactly the kind of overloaded term flagged above — it could mean a security pass, a style pass, a correctness pass, or all three, and the prompt gives the model no way to know which one you had in mind.

A revised version might read, as a system prompt: "You review pull requests for a Python backend team. For every PR, check for three things only: (1) logic errors that would cause incorrect behavior, (2) missing test coverage for new code paths, and (3) violations of the team's error-handling convention, which requires all I/O operations to be wrapped in a try/except that logs the specific failure. Do not comment on formatting, naming, or style — a separate linter already handles that, and duplicate comments waste the author's time. For each issue found, cite the specific line number, explain why it's a problem, and suggest a concrete fix. If you find no issues in a category, say so explicitly rather than staying silent, so the author knows that category was actually checked." Then, as user content for a specific call: the diff itself, and nothing else.

Notice what changed. Scope is now explicit and bounded — three categories, and an explicit instruction *not* to comment on a fourth category a naive read of "review" might have included. Success criteria are checkable — line number, explanation, concrete fix, for every issue raised. The persistent standards live in the system prompt, where they'll apply consistently across every PR this reviewer ever processes, while the thing that changes call to call — the diff — lives in user content. And an instruction that might look redundant at first glance ("if you find no issues, say so explicitly") is actually closing a specific ambiguity: silence on a category could mean "checked, found nothing" or "didn't check," and those are very different signals to an author relying on the review.

This is the pattern worth internalizing: every fix in that revision traces back to one of the principles in this lecture, and none of the fixes required exotic techniques — just the discipline of noticing where the original prompt was relying on inference and replacing that reliance with an explicit statement.

**Why it matters for the exam:** Expect scenario questions that present a prompt with a specific failure mode — inconsistent output length, occasional missed edge cases, wrong assumptions about audience — and ask you to identify what structural weakness in the prompt caused it. The pattern to look for is almost always some form of implicit reliance: a success criterion that was assumed rather than stated, a scope boundary left to inference, or system-level behavior improperly mixed into user content. Being able to name the specific structural gap, not just recognize that "the prompt could be better," is what these questions are testing.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Intro](01-module-intro) | [Next: Structure Is Leaking Into Prompts! &rarr;](03-structure-is-leaking-into-prompts) |
|:---|---:|

<!-- prevnext:end -->
