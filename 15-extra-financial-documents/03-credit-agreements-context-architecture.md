---
title: Credit Agreements - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Financial Documents"
nav_order: 3
duration: "7:14"
---

# Credit Agreements - Context Architecture
{: .no_toc }

*Video duration: 7:14*

Once the workflow shape is settled — deal-type classification, template-driven extraction, checklist review — the next architectural question is what actually goes into the context window at each step. For a fifty-to-two-hundred-page legal document, "just paste in the whole thing every time" is neither efficient nor, past a certain length, even the right way to get accurate answers, so credit agreement context architecture is really about disciplined scoping: giving the model exactly the material a given step needs, correctly labeled, and nothing more.

## The three kinds of material in context

A credit agreement system typically has three distinct categories of content competing for context space, and treating them as genuinely different — rather than one undifferentiated blob of "document text" — is the core design move.

**The agreement itself.** This is the primary source: the specific credit agreement under review, or the relevant portion of it for the current task. It is the only content in context that's authoritative about *this* deal, and everything else in context exists to help interpret it, not to substitute for it.

**The defined-terms glossary.** Every credit agreement opens with an "Definitions" article — often twenty or more pages on its own — that fixes the meaning of every capitalized term used for the rest of the document: *Indebtedness*, *Permitted Liens*, *Consolidated EBITDA*, *Change of Control*. These definitions are frequently bespoke to the deal (a fund's definition of *Consolidated EBITDA* might add back stock comp and one-time restructuring costs in ways that differ meaningfully from another deal's definition of the same term), and a covenant several dozen pages later is unreadable in isolation without them. That makes the glossary a load-bearing, always-needed piece of context whenever the model is reasoning about any covenant, representation, or event of default — not an optional reference to consult only if asked.

**Relevant precedent clauses.** When the task is deviation flagging or comparative Q&A ("is this leverage covenant tighter than what we normally accept?"), the model also needs a small set of comparison clauses pulled from the firm's standard paper or a precedent library — not the entirety of every past deal the firm has ever done, but the specific clause type relevant to the comparison at hand. This category is retrieved rather than always-loaded, which is the subject of the next lecture; the point here is simply that it's architecturally distinct from the agreement itself and from the glossary — it's comparison material, not primary source material, and it needs to be labeled as such so the model never confuses "what the standard paper usually says" with "what this specific agreement says."

| Context category | Source | When it's in context |
|---|---|---|
| The agreement itself | The specific deal under review | Always — the primary, authoritative source for the task |
| Defined-terms glossary | The agreement's own Definitions article | Always — needed to interpret nearly any other section |
| Precedent clauses | Firm's standard paper or precedent library | Retrieved — only for deviation flagging or comparative Q&A |

## Chunking and scoping a document this long

A full credit agreement, especially for a large syndicated facility, can run well past what's sensible to hold in a single working context alongside a glossary, precedent clauses, and the model's own reasoning — and even within a large context window, more irrelevant text is not free: it dilutes attention and increases the odds the model latches onto the wrong passage. The practical answer is to chunk by the document's own legal structure rather than by arbitrary token counts. A credit agreement's articles — Definitions, Representations and Warranties, Affirmative Covenants, Negative Covenants, Events of Default, Miscellaneous — are natural, meaningful chunk boundaries that a paralegal would recognize instinctively, and chunking along them (rather than, say, every 2,000 tokens regardless of where that lands) keeps each chunk semantically coherent: a chunk boundary that falls mid-covenant is far more dangerous than one that falls between two unrelated sections, because it can silently sever a threshold from the exception that qualifies it.

Given that structure, a given task pulls in only the chunks it actually needs, always paired with the glossary: extracting financial covenant thresholds needs the Negative Covenants article (plus Definitions); checking conditions to a draw needs the relevant conditions-precedent section; answering "what triggers mandatory prepayment" needs the mandatory prepayment provisions, which are sometimes in their own article and sometimes folded into affirmative covenants depending on the deal's drafting conventions — which is itself a reason the deal-type template from the previous lecture should specify *where* each field conventionally lives, so the retrieval step knows which chunks to pull without the model having to search blindly. The Definitions article is the one chunk that is close to a constant across nearly every task, precisely because so much of the rest of the document is unreadable without it — it functions less like retrieved content and more like a standing piece of context for the duration of work on a given agreement.

## Avoiding cross-references that silently break

Credit agreements are dense with internal cross-references — "as defined in Section 1.01," "subject to the exceptions set forth in Section 7.03(b)" — and a chunking strategy that isn't cross-reference-aware will happily hand the model a covenant that references an exception it never sees. Two practical mitigations matter here. First, when a chunk is pulled into context, a lightweight pass resolves its direct cross-references and pulls the referenced section (or at minimum, its defined term) along with it, rather than leaving the model to either hallucinate the referenced content or silently ignore that a qualification exists. Second, every extracted answer should carry its **section-level provenance** — which article and section the model drew a given fact from — so that a human reviewer checking the output can jump straight to the source language rather than re-reading the whole document to verify a single number. That provenance requirement is what makes the retrieval and citation architecture in the next lecture possible at all: it isn't a nice-to-have layered on afterward, it's a property the context has to be structured to support from the start.

**Architecture Notes:** This is Context-layer work in its purest form — deciding what's in the working set (the agreement, the glossary, retrieved precedent), how it's chunked (by legal structure, not token count), and how provenance is preserved so a human can trace any claim back to its source. The glossary's near-constant presence versus precedent clauses' retrieved, task-specific presence is itself a data-provenance distinction: the model must never treat "what standard paper says" as "what this document says," and correct labeling in context is what keeps that boundary from blurring.

<!-- prevnext:start -->

---

| [&larr; Previous: Credit Agreements - Local Workflows](02-credit-agreements-local-workflows) | [Next: Credit Agreements - RAG & Retrieval Architecture &rarr;](04-credit-agreements-rag-and-retrieval-architecture) |
|:---|---:|

<!-- prevnext:end -->
