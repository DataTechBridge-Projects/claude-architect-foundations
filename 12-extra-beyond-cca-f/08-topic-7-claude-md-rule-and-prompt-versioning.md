---
title: "Topic 7: CLAUDE.md, Rule and Prompt Versioning"
parent: "Extra Module: Beyond the CCA-F"
nav_order: 8
duration: "0:27"
---

# Topic 7: CLAUDE.md, Rule and Prompt Versioning
{: .no_toc }

*Video duration: 0:27*

A CLAUDE.md file, a system prompt, a tool description, and a rule embedded in a workflow step are all production logic, in exactly the same sense that a function in your codebase is production logic — they determine system behavior, they can contain bugs, and they can regress. Yet teams that would never let a code change ship without review routinely let a prompt tweak go straight to production because "it's just text." That asymmetry is the gap this topic addresses: these artifacts belong in version control, with the same discipline applied to any other code — reviewed changes, a readable diff, a changelog explaining what changed and why, and, critically, the ability to roll back cleanly when a change turns out to be a regression rather than an improvement.

The piece most teams miss even after they start versioning prompts is the *link* between a change and its evaluation. Versioning without evaluation just gives you a detailed history of when things got worse — it doesn't stop them from getting worse in the first place. A prompt or CLAUDE.md change should run against the eval suite described in Topic 3 before it ships, not after a user reports a problem, and the commit message or pull request should state what behavior the change was meant to fix or improve, so that a future regression has a paper trail to follow back.

Without that discipline, prompt and rule edits become organizational folklore: "someone changed something last sprint and now it's worse," with no record of what, who, or why, and no fast way to identify which of a dozen small edits actually caused the regression. Versioning turns that folklore into a diffable, bisectable history — the same benefit code review gives you, applied to the artifacts that increasingly *are* the system's logic in an agentic architecture.
