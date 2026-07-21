---
title: Key Distinctions - The Tool Layer
parent: The Tool Layer
nav_order: 14
duration: "5:12"
---

# Key Distinctions - The Tool Layer
{: .no_toc }

*Video duration: 5:12*

A condensed pass through the module's highest-yield either/or distinctions — the kind of pairs the exam likes to test directly against a scenario.

**Tool design as API design for a model, not a human.** The caller reading a tool's name, description, and schema is a reasoning model mid-inference with no ability to ask a clarifying question out of band. Every ambiguity left in a definition becomes a real, measurable rate of wrong calls — it isn't a documentation nicety, it's the interface.

**Selection failure versus model failure.** When a model calls the wrong tool among similar options, the productive diagnosis is almost always "what distinguishing information was missing from the definitions," not "the model should have known better." Tool descriptions should be written with the entire tool set in view, since selection is inherently relative to the alternatives available.

**Too many overlapping tools versus too few overloaded ones.** Narrow, near-duplicate tools multiply the pairwise disambiguation burden as a set grows. One tool with a tangled, cross-dependent parameter space pushes the same burden into argument-filling instead of selection. Match granularity to how distinct and how frequently-combined the underlying operations really are — there's no fixed right number of tools.

**Structure versus prose within a schema.** Default to structured, validatable fields — types, enums, ranges. Reserve free-text prose for fields where genuine judgment or open-ended variation is needed and enumerating the space in advance would constrain more than it would help.

**Multiple pathways versus a single rigid path.** A tool set built around one assumed route to a goal breaks the moment the world doesn't match that assumption. Offering more than one way to reach an outcome — search alongside direct lookup, a broad tool alongside a narrow one — is resilience, not redundancy.

**Retryable versus non-retryable failure.** This is the single most load-bearing signal an error response carries. Collapsing it into one generic failure message leaves the model with genuinely insufficient information, since the correct response to each is the opposite of the other: try again versus change approach entirely.

**Designed error messages versus raw stack traces.** A stack trace is optimized for a human reading source code; a tool error is a message to a reasoning system. Designed errors state what went wrong, whether it's retryable, and ideally what a valid alternative looks like — raw exception passthrough does none of this and leaks implementation detail besides.

**Harness-level recovery versus model-level recovery.** Mechanical, clearly-transient failures are often better handled by the harness retrying silently than by spending a full model turn reasoning through an obvious retry. Reserve the model's judgment for the ambiguous middle: modified retries, fallback tool choices, and deciding when to stop.

**Retry versus fallback versus escalation.** Retry when the failure is transient or a corrected argument would plausibly fix it. Fall back to a different tool when one exists and the primary path is a dead end. Escalate to a human when retries and fallbacks are exhausted, when the task's underlying assumptions appear wrong, or when continuing unsupervised is itself the risk.

**MCP server versus MCP client.** A server knows about one external system and exposes tools, resources, and prompts through the protocol; a client (built into Claude Code or any Claude-based application) discovers and connects to servers, making their tools available to the model exactly like any other tool. MCP standardizes the connection and discovery mechanism — it doesn't change tool-design principles, which apply identically to MCP-exposed tools.

**MCP governance: trust, scope, and audit.** A connected MCP server is functionally a dependency with real system access. Which servers are trusted (vetting process), what scope of access a connected server gets (least-privilege credentialing, not just hoping the model behaves), and whether actions can be reconstructed after the fact (audit logging) are organizational governance questions that grow directly with the size of a connected tool ecosystem.

**Schema validity versus operational safety.** A schema enforces that a call is well-formed; it says nothing about whether the underlying action is safe to allow. `Bash`'s narrow schema and broad real-world capability is the clearest example — safety comes from permissions, sandboxing, and scoping around the tool, not from the schema itself.

**Granularity matched per capability, not applied as a global policy.** Claude Code's own tool set splits file operations three ways, search two ways, and leaves shell execution as one broad tool — because each capability's own shape, not a single house rule, determines the right granularity.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Outro](13-module-outro) | [Next: Quiz Prep: The Tool Layer (20 Questions) &rarr;](15-quiz-prep-the-tool-layer-20-questions) |
|:---|---:|

<!-- prevnext:end -->
