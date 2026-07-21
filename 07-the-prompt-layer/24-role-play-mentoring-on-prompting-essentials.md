---
title: "Role Play: Mentoring on Prompting Essentials"
parent: The Prompt Layer
nav_order: 24
---

# Role Play: Mentoring on Prompting Essentials

**Priya (mentee):** I've got a prompt that classifies support tickets by urgency and routes them. It worked great in my demo — three examples, clean results. Now it's live and support is complaining it keeps calling billing questions "urgent" and missing actually urgent security reports.

**Dev (mentor):** Send me the three examples you used.

**Priya:** Sure — all three "high urgency" ones are billing disputes where the customer mentions a specific dollar amount they're owed.

**Dev:** There's your bug. None of your examples separate "urgent" from "mentions money." The model didn't learn your actual definition of urgency — it learned the thing all three examples happened to share.

**Priya:** But I never told it money equals urgent.

**Dev:** You didn't have to. It's pattern-matching across your examples, not reading your intentions. If every "high" example has a dollar figure and every "low" example doesn't, that correlation becomes part of the learned pattern whether you meant it or not. This is exactly the incidental-correlation trap from the few-shot lecture — the fix isn't fewer examples, it's examples that break the correlation on purpose.

**Priya:** So I add an urgent example with no dollar amount, and a non-urgent one that has one.

**Dev:** Right. A security report with zero financial language, marked high. A routine billing question that happens to mention an amount, marked low. Now "urgent" can't hide behind "mentions money" anymore — the model has to find the actual signal.

**Priya:** Should I also just... tell it not to use dollar amounts as a signal?

**Dev:** You can, and it won't hurt, but don't stop there. Explicit instructions help, examples teach by demonstration — you want both pulling the same direction, not one compensating for the other's gap. And while we're in here — what's your success criteria for "urgent" actually say?

**Priya:** Just "flag urgent tickets."

**Dev:** That's the second bug. "Urgent" is doing a lot of invisible work. Define it — active security exposure, a stated SLA breach, an outage affecting multiple customers, whatever your actual bar is — and write that down as an explicit, checkable criterion. Vague criteria produce vague, inconsistent classification, examples or no examples.

**Priya:** Okay. Last thing — how do I make sure I'm not just fixing what I noticed and missing what I didn't?

**Dev:** Batch test it. Build a set of twenty or thirty tickets that actually spans your production range — short ones, long ones, different registers, a few genuinely ambiguous ones — and run the whole set every time you touch the prompt. Don't eyeball one ticket and call it fixed. Look at the batch as a set; that's where you'll see if a fix helped everywhere or just patched the one case you were staring at.

**Priya:** And if I'm still not confident after that?

**Dev:** Then it's worth a real reviewer pass — a separate prompt whose only job is to double-check the classification against your written criteria, with no visibility into why the first prompt made its call. Not gentle, either — frame it to actively hunt for misclassifications, not confirm them. That costs you an extra call per ticket, so save it for the tickets your first pass flags as uncertain, not every single one.

**Priya:** So: fix the examples, write real criteria, test as a batch, and only add review where it's actually worth the cost.

**Dev:** That's the whole module in one sentence. Ship it, watch the batch metrics, and version the prompt so you can see exactly what changed when it gets better — or worse.

<!-- prevnext:start -->

---

| [&larr; Previous: Quiz Prep: The Prompt Layer (20 Questions)](23-quiz-prep-the-prompt-layer-20-questions) | [Next: The Context Layer &rarr;](../08-the-context-layer/) |
|:---|---:|

<!-- prevnext:end -->
