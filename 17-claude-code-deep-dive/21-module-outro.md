---
title: Module Outro
parent: Claude Code Deep Dive
nav_order: 21
duration: "2:00"
---

# Module Outro
{: .no_toc }

*Video duration: 2:00*

This module took a product the Workflow Layer had only sketched — CLAUDE.md exists, Skills exist, subagents are like Skills — and went all the way down to its actual mechanics. Start from where that arc began: Claude Code is the agentic loop from the Agentic Layer, instantiated as a specific harness with a curated tool set, a permission layer, and its own context-management discipline, not a different kind of thing from what the rest of this course already taught you to reason about.

Configuration turned out to be richer than a single file: three levels of `CLAUDE.md` that concatenate rather than override, a settings system right next door that resolves the opposite way — genuine precedence, deterministically enforced — and a family of mechanisms (subdirectory scoping, exclusion rules, sandboxing) for keeping both the right knowledge in and the wrong content out. The recurring lesson across all of it was the same one this course keeps returning to: anything delivered as context is advisory, no matter how many places it's stated; anything enforced by the harness is authoritative.

Extension mechanisms formed a deliberate ladder — slash commands for a saved prompt, Skills for a packaged procedure loaded on demand, subagents for genuinely isolated delegation, Plan Mode for gating an entire approach rather than one action at a time, and hooks for anything that has to be true mechanically, regardless of what the model decides. Each earns its place in a different, identifiable niche, and telling them apart under exam pressure comes down to the same handful of questions asked consistently: advisory or enforced, always-loaded or on-demand, inline or isolated.

Finally, the module moved from a human at a keyboard to production use — the Agent SDK for building your own application on the same engine, and non-interactive mode for running that same engine, unattended, inside a pipeline. Same loop, same tools, different driver.

Carry the pattern-matching skill this module built — not just the vocabulary — into the rest of your exam preparation.

<!-- prevnext:start -->

---

| [&larr; Previous: Claude Code in CI/CD: Non-Interactive Mode and the -p Flag](20-claude-code-in-ci-cd-non-interactive-mode-and-the-p-flag) | [Next: Key Distinctions - Claude Code Deep Dive &rarr;](22-key-distinctions-claude-code-deep-dive) |
|:---|---:|

<!-- prevnext:end -->
