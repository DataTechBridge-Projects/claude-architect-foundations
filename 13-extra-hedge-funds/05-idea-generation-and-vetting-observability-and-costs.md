---
title: Idea Generation/Vetting - Observability and Costs
parent: "Extra Module: Solution Architecture Patterns - Hedge Funds"
nav_order: 5
duration: "10:56"
---

# Idea Generation/Vetting - Observability and Costs
{: .no_toc }

*Video duration: 10:56*

A pipeline that screens, generates, and vets trade ideas can look healthy by every conventional software metric — low latency, no errors, high uptime — while quietly getting worse at the one thing that actually matters: producing ideas worth an analyst's time. Observability for this system has to be built around that distinction. Uptime and error rate tell you the system is *running*; they tell you nothing about whether it's running *well*, and the gap between those two things is where an idea generation system can degrade for months before anyone notices, because a subtly worse thesis still reads as fluent, confident prose.

## Tracking Accuracy and Drift

The central observability challenge here is that there's no ground truth available at the moment an idea is generated — you don't know whether a thesis was "correct" until the market has had time to prove or disprove it, which can be weeks or quarters out. That delay means this system needs two layers of quality signal: fast proxy metrics available immediately, and slow outcome metrics that close the loop later.

The fast proxies are things the system can measure at generation time: whether the vetting agent's citations actually resolve to real, current source documents (a basic but essential grounding check); how often the vetting agent flags a severe risk-factor conflict on ideas the generation agent scored as high-confidence, which is a useful tension signal — a healthy system should see this happen sometimes, and a system where it *never* happens is more likely to indicate the vetting agent has gone soft than that the generation agent has gotten flawless; and how often the vetting agent's search for contradicting evidence comes back thin versus substantive, tracked as a distribution over time rather than a single number, since a system whose contradicting-evidence searches are trending toward consistently thin results may be losing its ability to genuinely challenge theses even if nothing else looks broken.

The slow outcome metrics require closing the loop against what actually happened: for every thesis an analyst acted on (or explicitly logged as considered-and-rejected), tag it, and periodically — monthly or quarterly, not in real time — compare the system's original framing against the outcome. Did the catalysts the generation agent identified actually play out? Did the risks the vetting agent flagged turn out to be the ones that mattered, or did the thesis fail for a reason nobody flagged? This is slow, labor-intensive, and can't be automated away, but it's the only source of information that answers the question "is this system actually getting better or worse at its job," and it should feed directly back into refining the screening criteria, the retrieval strategy, and the risk-factor taxonomy rather than sitting in a report nobody revisits.

**Drift** deserves separate attention because it's the failure mode most likely to go unnoticed. Model behavior can shift after a provider-side model update, prompt or tool changes can subtly alter what gets surfaced, and the market regime itself changes in ways that make yesterday's risk-factor weighting stale. Concretely, this means tracking distributional metrics over time — the average confidence score the generation agent assigns, the rate of severe risk flags, the average length and specificity of vetting citations — and alerting on meaningful shifts in those distributions rather than only checking for outright errors. A generation agent that's quietly become more confident on average without any corresponding change in the quality of its evidence is a drift signal worth investigating even though nothing "broke."

## Human Override Rate as a First-Class Metric

The single most informative operational metric this system produces is the **human override rate** — how often, and in what direction, analysts disagree with the system's output. This needs to be tracked at a level of granularity beyond a single number: the rate at which analysts discard ideas the system scored as strong (a false-positive signal on the generation side), the rate at which analysts pursue ideas the vetting agent flagged as high-risk anyway (which isn't necessarily a system failure — sometimes a human correctly overrides a conservative flag — but is worth understanding in aggregate), and the rate at which analysts request additional retrieval or evidence the system didn't surface on its own (a direct signal about retrieval coverage gaps).

A rising override rate isn't automatically bad — it might mean analysts are engaging critically rather than rubber-stamping, which is the intended behavior. A *falling* override rate deserves more scrutiny than a rising one, because it can indicate either that the system has genuinely improved, or that reviewers are trusting it enough to review less carefully — a form of automation complacency that is especially dangerous in a domain where a fabricated-but-fluent citation can slip past a tired reviewer. Distinguishing those two explanations usually requires periodically sampling reviewed-and-approved ideas for a deeper audit, independent of the routine review process, precisely to check whether approvals are still substantive.

## Cost Management

Cost in this system is driven by three largely independent levers: model tier, caching, and the batch-versus-real-time split, and none of them should be treated as a single global setting — the right choice varies by stage.

**Model tiering** maps naturally onto the pipeline's three stages because they have very different accuracy-per-dollar requirements. The screening stage processes the highest volume of the lowest-stakes decisions — "is this worth a closer look" — and is the best candidate for a smaller, faster, cheaper model tier, since the cost of an occasional missed or over-included candidate is low and gets caught downstream anyway. The generation and vetting stages handle far lower volume but far higher-stakes reasoning, and are worth running on the strongest available model tier — this is exactly the kind of task where a cheaper model's occasional lapse in careful multi-step reasoning is costly in a way that a screening lapse isn't. Getting this allocation backwards — a strong model doing cheap screening, a weak model doing the actual vetting — is a common and expensive mistake.

**Prompt caching** is a natural fit for the stable pieces of context identified in the earlier context-architecture lecture: the risk-factor taxonomy loaded into the vetting agent, the system instructions and output-format specifications for each stage, and any standing few-shot examples used to keep thesis structure consistent. These change rarely and are reused across every invocation of their respective agent, making them close to ideal caching candidates — the savings compound directly with volume, since the screening stage alone may process hundreds of candidates a day.

**Batch versus real-time** is where the biggest cost lever in the whole system lives, because almost none of this pipeline actually needs to run in real time. Screening large volumes of overnight filings and news doesn't need sub-second latency — it needs to be done before the desk's morning review, which gives it a wide batch-processing window with correspondingly lower cost per unit of throughput. Generation and vetting on flagged candidates likewise don't need to happen the instant a flag fires; batching them into a scheduled run (or a short queue an analyst pulls from) rather than triggering each one synchronously both smooths cost and, as a side benefit, keeps the system from encouraging the kind of reflexive, ungrounded urgency that's already a risk in fast-moving markets. Real-time processing should be reserved for the rare case where an analyst is actively working a specific candidate interactively and wants the system to respond to a follow-up question immediately — a genuinely different usage pattern from the bulk screening and drafting work that makes up the overwhelming majority of the system's volume.

| Pipeline Stage | Volume / Stakes | Model Tier | Batch vs. Real-Time |
|---|---|---|---|
| Screening | High volume, low stakes | Smaller, cheaper tier | Batch (overnight, before morning review) |
| Generation | Low volume, high stakes | Strongest available tier | Batch / scheduled queue |
| Vetting | Low volume, high stakes | Strongest available tier | Batch / scheduled queue |

## Architecture Notes

Cost tiering that matches model strength to each stage's stakes, and batch processing for everything that doesn't need to be synchronous, are Workflow-layer decisions made concrete: not every step in a pipeline deserves the same investment. The human override rate as a primary observability signal is this use case's version of the Context layer's human-review-checkpoint discipline — tracking not just whether the checkpoint exists, but whether it's still functioning as genuine judgment rather than a rubber stamp.

<!-- prevnext:start -->

---

| [&larr; Previous: Idea Generation/Vetting - RAG.& Retrieval Architecture](04-idea-generation-and-vetting-rag-and-retrieval-architecture) | [Next: Trade Risk Management/Compliance - Overview &rarr;](06-trade-risk-management-and-compliance-overview) |
|:---|---:|

<!-- prevnext:end -->
