---
title: Structure Is Leaking Into Prompts!
parent: The Prompt Layer
nav_order: 3
duration: "0:19"
---

# Structure Is Leaking Into Prompts!
{: .no_toc }

*Read duration: 0:19*

As a prompt matures into something a production system depends on, it accumulates things that don't feel like "instructions" anymore: JSON schemas spelled out field by field, enumerated value lists, explicit delimiters marking where one section ends and another begins. That's structure leaking into the prompt — natural language alone can't reliably pin down a machine-parseable contract, so the schema migrates into the prompt text itself. The lesson: once a prompt carries that much structural weight, it has stopped being a request and **become an interface contract, deserving the same versioning discipline as an API definition**, not the casual editing habits of a first draft.

## A Concrete Example

Picture a support-ticket classification prompt that begins its life as a single sentence: "Classify this ticket as urgent or not urgent." Six months later, after the system has grown to route tickets across nine teams, handle three severity tiers, and feed a downstream dashboard that expects a specific JSON shape, that same prompt has quietly grown to include a full enumerated list of nine team names with routing rules for each, a three-value severity enum with explicit definitions distinguishing "high" from "urgent," delimiter tags marking where the ticket body starts and ends so it can't be confused with instructions, and a field-by-field JSON schema because the dashboard's parser breaks on anything else. None of that structure was designed in from the start — it accumulated, one edge case and one downstream requirement at a time, and at no point did anyone consciously decide "this prompt is now an interface contract." It just became one.

## The Fix Is to Externalize, Not Just Notice

Naming the leak is the first step, but the actual fix is usually to stop fighting it inside prose and instead move that structural weight into a place designed to hold it. The enumerated team list and severity values belong in a formal schema — ideally the input schema of a tool the model is directed to call, as covered in the lecture on achieving structured outputs — not embedded as a paragraph of prose the model has to re-parse and honor by convention on every single call. Once structure lives in a schema, it can be validated mechanically, diffed in code review, and versioned independently of the surrounding instructional language, which is a far more durable arrangement than a growing wall of enumerated text sitting inside a system prompt.

**Why it matters for the exam:** Expect a scenario where a prompt has accumulated schema-like content over time and you're asked to identify the underlying architectural issue. The correct diagnosis is that the prompt has outgrown plain instructional text and needs formal schema enforcement — not a rewrite with clearer prose.

<!-- prevnext:start -->

---

| [&larr; Previous: Prompting Best Practices](02-prompting-best-practices) | [Next: Few-Shot Prompting &rarr;](04-few-shot-prompting) |
|:---|---:|

<!-- prevnext:end -->
