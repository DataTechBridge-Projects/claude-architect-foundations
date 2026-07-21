---
title: Key Distinctions - The Workflow Layer
parent: The Workflow Layer
nav_order: 12
duration: "2:47"
---

# Key Distinctions - The Workflow Layer
{: .no_toc }

*Video duration: 2:47*

A tight reference list of the either/or calls this module keeps coming back to — the ones most likely to show up as the hinge of a scenario question.

- **`CLAUDE.md` vs. system prompt.** `CLAUDE.md` is project-scoped, team-owned, version-controlled alongside the code, and defines what the agent needs to know to work in *this* repo. A system prompt is harness-level, defines the agent's general role and behavior, and typically spans many projects.

- **`CLAUDE.md` vs. RAG/retrieval.** `CLAUDE.md` for small, stable, always-relevant knowledge loaded on every turn. Retrieval for large, dynamic, or query-specific knowledge that shouldn't be paid for in every session.

- **Skill vs. `CLAUDE.md`.** A Skill is procedural and loaded on demand for a specific, occasionally-needed task. A `CLAUDE.md` entry is declarative and loaded automatically, every session, regardless of task.

- **Skill vs. subagent.** A Skill loads instructions (and maybe tools) into the *current* agent's existing context — same agent, same model-call thread. A subagent gets its own context window and makes its own independent model calls, reporting back a result. Different mechanisms, even when the use case looks similar from outside.

- **Deterministic step vs. agentic step.** Deterministic if the correct behavior can be fully specified in advance, independent of the specific input (a schema check, a test run). Agentic if the correct behavior genuinely depends on judgment about the specific case in front of it. Complexity alone doesn't make a step agentic — only genuine ambiguity or context-dependence does.

- **Fixed pipeline vs. agent-controlled sequencing.** If the order of steps can be pinned down in advance, use a fixed pipeline (individual steps can still be agentic). If the right next step depends on what was just discovered, the sequencing itself needs agentic control.

- **Centralized convention vs. local convention.** Some workflow decisions (hard constraints, security-relevant gates) belong in an org-wide standard. Many others (report formats, review depth, which Skills exist) are legitimately local to one team or repo and shouldn't be forced into a single template.

- **Ad hoc Claude usage vs. CI/CD-integrated step.** Ad hoc is manual, invisible to the team, and inconsistent. CI/CD-integrated has a defined trigger, reproducible input, durable captured output, explicit gating, and the same operational rigor (logging, retries, rollback) as any other pipeline step.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Outro](11-module-outro) | [Next: Quiz Prep: The Workflow Layer (20 Questions) &rarr;](13-quiz-prep-the-workflow-layer-20-questions) |
|:---|---:|

<!-- prevnext:end -->
