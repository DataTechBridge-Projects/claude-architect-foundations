---
title: Module Outro
parent: The Workflow Layer
nav_order: 11
duration: "1:52"
---

# Module Outro
{: .no_toc }

*Read duration: 1:52*

This module moved a level below the Agentic layer's "who decides" question and a level above the individual prompt: workflow design is about how work gets configured and organized so that decisions get made consistently, without someone re-explaining the same context every session. You saw `CLAUDE.md` as the mechanism for standing, always-relevant project knowledge — automatic, persistent, and cheap, but easy to bloat if treated as a dumping ground rather than a curated set of durable facts. You saw Skills as the complementary mechanism for procedural knowledge that's real but only occasionally relevant, loaded into the *same* agent's context on demand rather than sitting in every session by default — and you saw exactly how that differs from a subagent, which gets its own context and its own model call entirely. You worked through sequencing a workflow and deciding, step by step, which parts are deterministic and which genuinely need judgment, and how that feeds back into the Agentic layer's single-agent-versus-pipeline-versus-multi-agent decision. And you saw how this extends into CI/CD, where a Claude-driven step stops being something a person remembers to run and becomes an automated, auditable, gated pipeline participant — held to the same operational standards as everything else in it.

The throughline across all of it is placement: knowing exactly which mechanism a given piece of knowledge, procedure, or process belongs in, and resisting the urge to force everything into whichever mechanism you reached for first. A `CLAUDE.md` bloated with procedures that should have been Skills, a Skill library padded with facts that should have been a couple of lines of standing context, a subagent spun up for a task a Skill would have handled more cheaply, a CI gate left undefined because nobody decided whether a finding should block or merely advise — each of these is the same underlying mistake wearing a different costume: reaching for the mechanism that's top of mind instead of the one the knowledge actually calls for. The lab made you practice catching that mistake before it shipped, not just recognize it in hindsight.

It's also worth naming what stayed constant underneath all the mechanism-swapping: every placement decision in this module reduced to the same handful of questions asked about a different kind of thing each time — how stable is it, how universally does it apply, does it need judgment or just consistency, and what happens if it's wrong. Those questions don't go away once you leave the Workflow layer; they resurface, reshaped, in how you write an individual prompt and in how you decide what belongs in a model's context window at all — which is exactly where the next two modules pick up.

Next up is the Prompt layer, where the focus shifts from *how work is organized* to *how intent gets communicated to the model in a single request* — the layer most people think they already understand, and the one this course will push hardest to deepen.

<!-- prevnext:start -->

---

| [&larr; Previous: Lab: Workflow Strategy Simulator](10-lab-workflow-strategy-simulator) | [Next: Key Distinctions - The Workflow Layer &rarr;](12-key-distinctions-the-workflow-layer) |
|:---|---:|

<!-- prevnext:end -->
