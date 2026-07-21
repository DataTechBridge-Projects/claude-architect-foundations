---
title: LPA/Subscription Agreements - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Financial Documents"
nav_order: 8
duration: "7:36"
---

# LPA/Subscription Agreements - Context Architecture
{: .no_toc }

*Video duration: 7:36*

Context architecture for subscription documents starts from a constraint that didn't exist in the same form for credit agreements: the system routinely processes many documents belonging to different, mutually confidential parties in the same batch run, sometimes close together in time or even concurrently. Getting the "what's in context" question right here is inseparable from getting the "what's kept *out* of context" question right, because the cost of the two mixing isn't a wrong extracted number — it's one investor's confidential commitment terms appearing in another investor's output.

## What's in context per document

For a single subscription document being processed, the working context is deliberately narrow: the specific investor's subscription agreement itself, the extraction template for that document type, and — when needed for a consistency check — the relevant provisions of the fund's governing LPA. That's close to the full list. Unlike credit agreements, there's generally no need for a broad defined-terms glossary specific to each subscription document, because subscription documents mostly operate by reference to definitions already fixed in the LPA rather than defining a large bespoke vocabulary of their own; where a subscription document does use a defined term, it's typically inheriting the LPA's definition, not creating a new one, so the LPA excerpt itself serves the same role the defined-terms glossary served for credit agreements.

The governing LPA deserves its own treatment as a distinct context category, because it plays two different roles depending on the task. For **extraction**, it's largely irrelevant — pulling an investor's name and commitment amount out of their subscription document doesn't require the LPA at all. For **consistency checking** — verifying that what an investor is agreeing to in their subscription document doesn't conflict with what the LPA actually permits or requires — the relevant LPA provisions become essential primary material, on par with the subscription document itself. That task-dependence is worth designing for explicitly rather than defaulting to "always include the LPA just in case," both because it's wasted context and cost on the (much more common) pure-extraction tasks, and because unnecessarily including material that isn't needed for a given task is itself a minor provenance risk — it makes it more likely the model conflates "what the LPA says in general" with "what this specific investor agreed to."

## Avoiding cross-contamination between investors

This is the architectural property that most distinguishes this module from the credit agreements one, and it has to be designed in at multiple levels, not assumed to fall out naturally from "processing documents one at a time."

**Per-document context isolation.** Each subscription document is processed in its own isolated context — its own model call, or its own clearly bounded turn in a session, with no residue carried over from the previous investor's document. This sounds obvious, but it's the single most important rule in the whole module, because the efficient thing to do from a caching and throughput perspective — batch many documents through a shared session to amortize setup cost — is exactly the thing that creates cross-contamination risk if the context isn't explicitly cleared or scoped between documents. The architecture has to treat "fast" and "isolated" as compatible goals achieved through explicit engineering (parallel isolated calls, or a session that's deliberately reset between documents), not as goals that are in tension and have to be traded off against each other.

{: .important }
> Per-document context isolation is the single most important rule in this module. Never batch multiple investors' documents through a shared session to save on setup cost without explicitly clearing or scoping context between them — that throughput shortcut is exactly what causes one investor's confidential terms to bleed into another's output.

**No shared working memory across investor records.** Where the system does need cross-document awareness — for instance, the exception-pattern detection described in the previous lecture, which looks across all the exceptions flagged in a closing — that aggregation should operate over already-extracted, already-anonymized-where-appropriate structured metadata (exception type, deviation category) rather than over raw investor content sitting side by side in a shared context. The distinction matters: "three investors in this closing flagged the same type of tax-status exception" is a safe, useful aggregate signal; "here is investor A's exact language sitting next to investor B's exact language in the same context window" is not something the architecture should ever produce as an incidental side effect of trying to detect that pattern.

**Output scoping matches input scoping.** Whatever access controls govern who can see a given investor's subscription document should govern who can see the system's output about that document, and the system's own retrieval and context-assembly logic should enforce the same boundary — a query about Investor A's terms should structurally be unable to pull in Investor B's document as supporting context, not merely be instructed not to via a system prompt. Relying on an instruction ("don't mention other investors' terms") to enforce a confidentiality boundary that could instead be enforced structurally, by simply never putting the other investor's data in context in the first place, is a category of mistake worth naming explicitly — **instructions are a weaker control than context that was never assembled to include the sensitive material at all**.

## Handling side letters in context

Investor-specific side letters — separately negotiated terms layered on top of the standard LPA and subscription document — belong in context alongside that specific investor's subscription document when the task concerns that investor, and nowhere else. A side letter is, in context-architecture terms, exactly the same category of concern as the subscription document itself: investor-specific, confidential, and never appropriate to pull into another investor's processing context. The one legitimate exception — comparing terms *across* side letters to assess MFN implications — is a genuinely different task with genuinely different context requirements, and this course treats it as its own use case for exactly that reason; it's covered in full in the MFN and side letter negotiation module later in this section, where the architecture explicitly opens up the cross-investor comparison that this module just spent its effort keeping closed.

**Architecture Notes:** This lecture is a case study in the Context layer's data-provenance and isolation principles applied under a hard confidentiality constraint: what's in the working set per document (subscription agreement, template, relevant LPA excerpt) has to be assembled so that structural isolation — not instruction alone — prevents one investor's terms from ever entering another's context. The task-dependent inclusion of the LPA (needed for consistency checks, irrelevant for pure extraction) is itself a reminder that "more context" is not a default good; it's a cost and a provenance risk that should be justified by the specific task.

<!-- prevnext:start -->

---

| [&larr; Previous: LPA/Subscription Agreements - Local Workflows](07-lpa-and-subscription-agreements-local-workflows) | [Next: LPA/Subscription Agreements - RAG & Retrieval Architecture &rarr;](09-lpa-and-subscription-agreements-rag-and-retrieval-architectu) |
|:---|---:|

<!-- prevnext:end -->
