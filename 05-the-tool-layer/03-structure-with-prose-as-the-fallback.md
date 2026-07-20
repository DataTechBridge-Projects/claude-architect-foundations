---
title: Structure with Prose as the Fallback
parent: The Tool Layer
nav_order: 3
duration: "0:20"
---

# Structure with Prose as the Fallback
{: .no_toc }

*Video duration: 0:20*

Default every field toward structure — enums over free text, typed values over strings that encode meaning implicitly — because structure is what makes a call validatable before execution. Reach for a free-text prose field only where the content genuinely requires judgment or open-ended variation a fixed schema can't anticipate: a commit message, a customer-facing explanation, a search query's phrasing. The test is whether enumerating the valid space in advance would actually help the model or just constrain it into unnatural, ill-fitting choices. When in doubt, structure first, prose only where rigidity would clearly make the tool worse.

Take a concrete `issue_refund` tool as an example of the split in practice. The `amount` field should be a typed number, not a string the model formats as "$45.00" — a typed field can be range-checked before the call ever reaches a payment system. The `reason` field is a better fit for an enum — `["defective", "wrong_item", "customer_changed_mind", "other"]` — than free text, because downstream reporting almost certainly needs to bucket refunds by reason, and a model asked to pick from a fixed list will do so more reliably than one asked to phrase a category from scratch. But a `customer_note` field genuinely benefits from prose: there's no fixed set of things a support agent might need to say to a customer, and forcing that into an enum would either explode into an unmanageable list or silently drop information the situation actually called for.

The deeper reason to default toward structure isn't just tidiness — it's that structured fields are the ones a system can validate, log, and reason about *before* anything executes, the same way a strict schema catches a malformed API call before it reaches a server. A prose field is opaque until read; it can't be range-checked, can't be matched against an enum, and can't be mechanically distinguished from an injection attempt riding along inside it. That's a real cost, which is exactly why this lecture frames prose as a *fallback* rather than a co-equal design option: reach for it only when the alternative — cramming genuine judgment into a rigid field — would produce worse results than the risk of leaving a field unconstrained. This is the same instinct behind the enum discussion in the tool-definition lecture earlier in this module: an enum both validates and documents in one move, and every field you can legitimately convert to one is a field you no longer have to trust prose to get right.
