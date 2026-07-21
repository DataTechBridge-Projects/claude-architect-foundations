---
title: Data Provenance and Conflicts
parent: The Context Layer
nav_order: 12
duration: "19:22"
---

# Data Provenance and Conflicts
{: .no_toc }

*Read duration: 19:22*

A context window, read on its own terms, is just text — a sequence of statements with no inherent marker of how much any of them deserve to be trusted. "The customer's account was closed in March" and "the customer's account is currently active" can both appear in the same window, phrased with identical confidence, and nothing about the sentences themselves tells the model which one to believe, or whether they're even actually in conflict once you know where each came from. This lecture is about the discipline that fills that gap: **data provenance** — tracking where a piece of context came from, how it was obtained, how current it is, and how much it should be trusted — and what to do when two pieces of context that both carry some claim to authority disagree.

## Why provenance has to be explicit

The default state of a context window is that provenance is implicit at best and invisible at worst. A fact retrieved from an authoritative internal database and a fact pulled from an old email thread can end up looking identical once they're both rendered as plain text in the window — same font, so to speak, same grammatical confidence, same apparent status as "something currently known." Nothing about the mechanics of a context window automatically preserves the distinction between "this came from a system of record, verified thirty seconds ago" and "this came from a user's offhand remark two weeks ago and was never checked." If that distinction matters — and for any consequential decision, it does — someone has to deliberately design it into the system, because it will not survive by default.

This is the same principle that ran through the error-propagation lecture, now generalized: unmarked uncertainty is dangerous, and provenance is the mechanism that keeps uncertainty (and confidence) marked rather than allowing everything in context to flatten into undifferentiated, equally-trusted assertion. A context window with good provenance discipline doesn't just contain facts — it contains facts *plus metadata about those facts*, and that metadata is what allows every downstream judgment call in this module, from re-verification to human review design, to actually function.

## What good provenance metadata looks like

At minimum, useful provenance tracking attaches three things to a consequential piece of information: **source** (where did this come from — a specific tool call, a specific document, a user statement, an inference the agent itself made), **timestamp or currency** (when was this true, and how likely is it to still be true), and **trust level** (how authoritative is this source, and was it independently verified or simply accepted as stated).

**Source** matters because different sources carry structurally different reliability profiles, independent of anything about the specific claim. A number pulled directly from a query against a system of record carries a different kind of authority than the same number appearing in a user's paraphrase of what they remember being told, even if both happen to be correct in a given instance — the first is verifiable and reproducible, the second is secondhand and subject to the ordinary distortions of human memory and communication. Knowing which one you're looking at changes how much independent weight the claim deserves.

**Timestamp or currency** matters because facts have a shelf life that varies enormously by type. An organization's name rarely changes; an inventory count can be stale within minutes; a person's job title can be accurate for years and then wrong overnight. A piece of context without any sense of when it was true, or how volatile that kind of fact typically is, leaves the model unable to judge whether staleness is even a risk worth considering — this connects directly to the stale-context ambiguity pattern covered in the escalation lecture, and provenance tracking is the general mechanism that makes staleness visible before it becomes a silent failure.

**Trust level** matters because not all sources — even accurate ones — carry the same epistemic weight, and this is where provenance tracking has to capture something more nuanced than a binary "verified or not." A figure independently confirmed against two agreeing sources deserves more confidence than the same figure taken from a single unconfirmed source, even if both ultimately turn out correct; a conclusion the agent itself inferred deserves a visibly different trust level than a fact a tool returned directly, because an inference carries the compounded risk of every reasoning step that produced it, where a direct lookup doesn't.

| Dimension | Question it answers | Example |
|---|---|---|
| Source | Where did this come from? | A specific tool call, document, user statement, or the agent's own inference |
| Timestamp/currency | When was this true, and how likely is it still true? | "Verified 12 minutes ago" versus a two-week-old email |
| Trust level | How authoritative is this, and was it independently verified? | Independently confirmed, stated but unconfirmed, or inferred |

None of this needs to be an elaborate formal system in every case — for a lightweight task, a simple tag distinguishing "verified," "stated but unconfirmed," and "inferred" attached to key facts captures most of the value. What matters is that the distinction exists *somewhere* explicit, rather than being left for a reader (human or model) to guess at from tone and phrasing alone.

## Detecting and resolving conflicts

Provenance tracking earns most of its keep at the exact moment two pieces of context disagree, because that's precisely when a reader needs a principled way to choose between them rather than picking arbitrarily or averaging two things that shouldn't be averaged.

**Recognize the conflict as a conflict, not just a nuance.** The first failure mode is not noticing that two pieces of context actually contradict each other, especially when they're phrased differently enough, or separated by enough intervening text, that the contradiction isn't visually obvious. This is a direct extension of the ambiguity-recognition problem from earlier in the module: a model (or a human skimming a long document) can process both statements individually without ever explicitly registering that they can't both be true, particularly under the same fluency-over-accuracy pull discussed in that lecture. Designing prompts and workflows that explicitly instruct an agent to check for and flag contradictions — rather than assuming contradictions will be self-evident — is a meaningful part of getting this right.

**Weigh, don't just pick the most recent or the most confident-sounding.** Once a conflict is recognized, the naive resolutions are both tempting and both frequently wrong. "Trust whichever source is more recent" is a reasonable default for facts that are genuinely volatile (an inventory count, an account status) but a bad rule for facts that are stable and where the more recent source is actually a less reliable restatement of the same underlying truth (a user's imprecise recollection of a policy is not more authoritative than the policy document itself, no matter which one entered context more recently). "Trust whichever source sounds more confident or more detailed" is worse still, because confidence of phrasing has no necessary relationship to accuracy — an agent's own unverified inference can easily be phrased more confidently than a hedged, honest report from a reliable source. Real resolution requires weighing source authority, currency, and independent verifiability together, which is exactly why all three need to be tracked as separate dimensions rather than collapsed into one.

{: .important }
> Neither **"trust the most recent source"** nor **"trust the most confident-sounding source"** is a universal rule for resolving conflicting context — recency is a good tiebreaker only for genuinely volatile facts, and confident phrasing has no necessary relationship to accuracy. Weigh source authority, currency, and verifiability together, and escalate when they don't clearly agree.

**Escalate the conflict when the resolution rule doesn't clearly apply.** Not every conflict resolves cleanly using source, currency, and trust level alone — sometimes two similarly authoritative, similarly current sources genuinely disagree, and no amount of weighing settles it. This is exactly the kind of situation the escalation lecture's granular-response framework was built for: the correct move is not to arbitrarily pick one, but to surface the specific conflict, with both sources and their provenance attached, either as a flagged assumption if the stakes are low or as a targeted clarifying question or escalation if they're not. A well-designed system treats an unresolvable conflict as its own distinct decision point, not as a nuisance to be silently smoothed over on the way to producing an answer.

## Provenance across summarization and compaction

Provenance tracking has a direct and important interaction with the summarization and compaction techniques covered earlier in this module, and it's worth stating plainly: summarization is one of the most common places provenance quietly gets lost. A careless summary compresses "the refund amount, confirmed against the current order record at 2:14pm, is $190" down to "the refund amount is $190" — technically shorter and technically preserving the number, but stripping away exactly the metadata that would let a later turn, or a human reviewer, judge how much to trust that number if anything about the situation later seems off.

The fix isn't to never summarize provenance-bearing facts — that would defeat the purpose of summarization entirely — it's to treat provenance as part of what a good summary has to decide whether to keep, the same deliberate way it decides whether to keep a caveat or a rejected alternative. For genuinely load-bearing facts — the ones a downstream irreversible action will depend on — provenance should survive compression even when surrounding detail doesn't. A useful discipline: before compacting a fact away, ask not just "do I still need this number" but **"will whoever reads this later still be able to tell how much to trust it."** If the answer is no, the compaction was too aggressive for that particular fact, even if it was appropriately aggressive for everything around it.

## Provenance as the connective tissue of this module

Stepping back, provenance tracking isn't really a standalone technique sitting alongside the others in this module — it's the substrate that makes several of them work at all. Re-verification (from the error-propagation lecture) depends on being able to identify which facts are unverified in the first place. Escalation and ambiguity handling depends on recognizing when context is genuinely insufficient rather than just sparse, which is a provenance judgment as much as a content judgment. Human review checkpoints depend on being able to show a reviewer not just what the agent concluded but how confidently and on what basis, which is provenance rendered for a human reader instead of a model. Strip provenance out of any of these, and what's left is a system that can only ever treat everything in its context window as equally, uniformly true — which is precisely the condition that makes ambiguity invisible, escalation miscalibrated, and error propagation almost inevitable.

## A worked example of weighing, not picking

Return to the account-status conflict from earlier — a system-of-record lookup says a customer's account is active, while a two-week-old email in the conversation says the customer requested cancellation — and walk through what actually weighing the sources looks like rather than just naming the principle.

Start with source authority. The system-of-record lookup is, definitionally, the operational source of truth for account status; that's what the system is for. The email is a secondhand report of an *intention* the customer expressed, not a record of the cancellation actually being processed. Authority alone already tilts toward the lookup — but not conclusively, because the lookup could simply be stale or the cancellation request could have been lost or mishandled somewhere in the two weeks since.

Next, currency. The lookup is current as of whenever it was run — check the timestamp attached to it. If it was run this session, moments ago, it reflects the present state far more reliably than a two-week-old email reflects anything about the present, since account status is exactly the kind of fact that changes and a stated intent from two weeks ago says nothing about whether that intent was ever fulfilled.

Finally, verifiability. Is there a way to check whether the cancellation request in the email actually resulted in an action — a ticket number, a confirmation reference, a status field the email itself might mention? If the email includes a cancellation confirmation number, that's independently checkable against a different system of record, which would settle the conflict cleanly. If it's just the customer's own words with nothing to verify against, it remains an unconfirmed claim of intent, not a confirmed fact of current status.

Putting these together, the defensible conclusion in this specific case is: trust the current lookup as the account's actual status, but treat the cancellation request as unresolved business that needs to be checked, not ignored — surfacing something like "account currently shows active; a cancellation request from two weeks ago doesn't appear to have been processed — confirm before proceeding" is a materially better outcome than silently picking either source and moving on. Note what made this resolvable at all: each source carried enough attached metadata — a timestamp on the lookup, an approximate date on the email — to actually weigh them. Strip that metadata out through careless summarization, and this same conflict becomes unresolvable, forcing exactly the kind of arbitrary pick this section warns against.

**Why it matters for the exam:** Provenance and conflict-resolution questions on the CCA-F typically present two contradicting pieces of information in a scenario and ask what the agent should do. The wrong answers are almost always "pick the more recent one" or "pick the more detailed/confident-sounding one" presented as if either were a universal rule. The right answer requires actually reasoning about source authority and verifiability specific to that scenario — and, when the scenario doesn't give you enough to resolve it cleanly, recognizing that surfacing the conflict rather than silently resolving it is itself often the correct choice.

<!-- prevnext:start -->

---

| [&larr; Previous: Failure Affordance Varies By Use Case](11-failure-affordance-varies-by-use-case) | [Next: Putting the "Provenance" in Data Provenance &rarr;](13-putting-the-provenance-in-data-provenance) |
|:---|---:|

<!-- prevnext:end -->
