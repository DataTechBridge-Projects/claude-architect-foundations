---
title: Different Workflows, Similar Principles
parent: The Tool Layer
nav_order: 11
duration: "0:25"
---

# Different Workflows, Similar Principles
{: .no_toc }

*Video duration: 0:25*

A coding agent's tools, a customer-support agent's tools, and a data-processing pipeline's tools look nothing alike on the surface — files and shell commands versus order lookups and refund actions versus record transforms and validation checks. But every principle in this module applies unchanged across all three: names and descriptions must disambiguate among neighbors, errors must be specific and actionable, granularity must match how distinct the underlying operations really are, and high-blast-radius actions need the same scrutiny regardless of domain. The domain changes; the design discipline doesn't.

Put two of those tool sets side by side and the parallel becomes concrete. A coding agent choosing between `read_file` and `search_files` is solving exactly the same selection problem as a support agent choosing between `get_order_by_id` and `search_orders_by_customer` — in both cases the model needs the description to make clear which one applies when it has an exact identifier versus only partial information, and in both cases a vague pair of descriptions produces the same category of misselection regardless of whether the domain is a filesystem or an order database. Likewise, a `delete_file` tool and an `issue_refund` tool are both high-blast-radius, hard-to-reverse actions that deserve the same treatment covered in this module — an honest description of consequences, tight scoping, and infrastructure-level safeguards rather than reliance on the model simply choosing not to misuse them.

What genuinely does differ across domains is the *calibration* of blast radius and the *shape* of the recurring failure modes — a coding agent's riskiest tool might corrupt a repository, while a support agent's riskiest tool might move real money, and those carry different review bars, different rollback options, and different regulatory exposure even though the underlying design discipline treating them as "high blast radius, gate accordingly" is identical. A data-processing pipeline's tools tend to fail more often on granularity than on naming, because transforms and validations are frequently reasonable to bundle or split depending on pipeline stage, whereas a support agent's tools tend to fail more often on selection ambiguity, because customer language rarely maps cleanly onto a single intended tool. Recognizing which failure mode a given domain is more prone to is itself a transferable skill — but the fix, in every case, comes from the same toolbox this module has been building: sharper names, more honest descriptions, better-matched granularity, and infrastructure that doesn't depend on the model getting it right every time.
