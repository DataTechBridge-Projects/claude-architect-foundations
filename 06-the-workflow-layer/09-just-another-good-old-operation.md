---
title: Just Another Good, Old Operation
parent: The Workflow Layer
nav_order: 9
duration: "0:22"
---

# Just Another Good, Old Operation
{: .no_toc }

*Read duration: 0:22*

A Claude-driven CI step is still a pipeline step, and it doesn't get a pass on the discipline every other pipeline step earns the hard way. It needs logging, so a bad output is debuggable after the fact. It needs retries and timeouts, so a transient failure doesn't silently drop a check. It needs a rollback or fallback path, so a bad generated commit or a stuck job doesn't block the whole pipeline. **Treating an agentic step as "special" and exempting it from that rigor is exactly how it becomes the least reliable part of the system.**

Picture a concrete failure: an automated code-review step calls out to Claude on every PR, but the pipeline was wired up quickly, without the same care given to the deployment step next to it. One day the model call hangs — a transient network issue, a rate limit, doesn't matter which — and because no timeout was configured, the job sits there indefinitely instead of failing fast. Every PR behind it queues up waiting for a check that will never finish. A deterministic lint step with the same hang would have been caught in code review the moment someone proposed shipping it without a timeout; the agentic step got a pass simply because it felt newer and less like "real" infrastructure. That asymmetry is the actual risk, not the model call itself.

This is the same discipline the CI/CD Integration lecture lays out in more depth: defined triggers, reproducible input, durable output, explicit gating, and now — the operational layer underneath all of that — the boring reliability engineering every pipeline step needs regardless of what's running inside it. A retry policy for a flaky agentic call is no different in kind from a retry policy for a flaky network-dependent test; a fallback path for a stuck Claude-driven step (skip and flag for manual review, rather than hang the whole pipeline) is no different in kind from a fallback for any other step that can occasionally fail to complete. The "two-key" pattern from that lecture — Claude's output checked by a deterministic validator before a human ever needs to look at it — only works if the operational scaffolding around it (logging, timeouts, retries, rollback) is solid enough that the check itself is trustworthy. Skip the plumbing, and the fanciest gating logic in the world is validating output from a step that might not even have run to completion.

<!-- prevnext:start -->

---

| [&larr; Previous: CI/CD Integration](08-ci-and-cd-integration) | [Next: Lab: Workflow Strategy Simulator &rarr;](10-lab-workflow-strategy-simulator) |
|:---|---:|

<!-- prevnext:end -->
