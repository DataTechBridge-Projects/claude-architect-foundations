---
title: Splitting When Something Changes
parent: The Agentic Layer
nav_order: 11
duration: "0:26"
---

# Splitting When Something Changes
{: .no_toc }

*Video duration: 0:26*

A practical rule of thumb for where to place a decomposition boundary: split when something changes. If the next piece of work needs a different tool set than the current one, a different body of data or context than what's already loaded, or a different reviewer (a human approval where none was needed before, or a different subagent's specialization), that's a strong signal a step boundary belongs there. Steps that share tools, data, and reviewer usually belong together; steps that don't, usually don't — regardless of how naturally they seem to follow one another in the goal's original description.

Take a concrete workflow: "research a topic, draft a report, and publish it." On the surface this reads as one continuous task, but run the three-part test against it and the boundaries fall out cleanly. Research needs web-search and document-reading tools; drafting needs none of those but does need a large working context of everything research produced; publishing needs neither of those but does need a completely different tool (a CMS or file-write API) and, typically, a human approval gate that the earlier two steps didn't require at all. Each transition changes at least one of tools, data, or reviewer — so each transition is a legitimate split point, independent of the fact that the three phases read as one sentence in the original goal description.

| Step | Tools | Data | Reviewer |
|---|---|---|---|
| Research | Web search, document reading | Source material being gathered | None |
| Draft | None of the research tools | Full context of research findings | None |
| Publish | CMS / file-write API | The finished draft | Human approval gate |

The reverse case is just as instructive: two steps that look separate in a task description but share tools, data, and reviewer usually don't need a boundary at all. "Read the config file, then check whether the value is valid" doesn't warrant being split into two subagents or two workflow stages — both parts use the same tool (a file read), operate on the same small piece of data, and need no distinct reviewer. Splitting there would just add the coordination overhead described in the multi-agent coordination and "think of scaling" lectures without buying any of the benefits — no context isolation worth having, no parallelism opportunity, no specialization that actually differs.

This heuristic is also a useful check against splitting work in a way that mirrors an org chart or a project plan rather than the actual computational shape of the task. A step boundary that exists because a document happened to have three headings, rather than because the tools, data, or reviewer genuinely changed, is a sign the decomposition is being driven by the task's narrative structure instead of its architecture — and it's the kind of distinction the exam is likely to probe with a scenario that reads naturally as several steps but doesn't actually warrant splitting into several agents or workflow stages.

<!-- prevnext:start -->

---

| [&larr; Previous: Workflow Decomposition](10-workflow-decomposition) | [Next: Workflow Safeguarding &rarr;](12-workflow-safeguarding) |
|:---|---:|

<!-- prevnext:end -->
