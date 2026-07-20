---
title: Topic 3: Agent Performance Evaluation
parent: Extra Module: Beyond the CCA-F
nav_order: 4
duration: "0:20"
---

# Topic 3: Agent Performance Evaluation
{: .no_toc }

*Video duration: 0:20*

"It seems to work" isn't an evaluation strategy, and it's the single most common failure mode in teams shipping their first agentic system: someone tries a handful of prompts, likes what they see, and ships. That approach can't tell the difference between a genuine improvement and a lucky sample, and it has no way at all of catching a regression somewhere else in the system.

Serious agent evaluation starts with a held-out set of representative tasks — inputs pulled from real usage where possible, covering the range of cases the system actually has to handle, each paired with a known-good outcome or a clear rubric for what "good" looks like. Scoring those tasks depends on what kind of correctness you're checking: exact-match or structural checks work well when the output has a verifiable right answer (did it extract the correct field, call the correct tool, produce valid JSON), while LLM-as-judge scoring — using a separate model call to grade output against a rubric — is often the more practical option when correctness is fuzzier, like tone, completeness, or reasoning quality. The Prompt Layer's principle of not letting a pass grade its own homework applies directly here: the judge should be a distinct, independently-prompted pass, not the same call that produced the output.

The part that actually matters in practice isn't scoring one run — it's catching regressions over time. Every prompt tweak, tool change, or model upgrade is a chance to fix one failure mode while silently introducing another, and without a suite you re-run on every change, you won't know which happened until a user hits it in production. Treat your eval suite exactly the way you'd treat an automated test suite for code: version it alongside the prompts and tools it's evaluating, run it in CI before any of those artifacts ship, and trust its aggregate results over your own impression from skimming a few transcripts — intuition is a fine way to generate hypotheses, but a terrible way to confirm them.
