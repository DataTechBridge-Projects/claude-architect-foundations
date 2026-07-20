---
title: The Infrastructure for Tool Usage
parent: The Tool Layer
nav_order: 9
duration: "0:21"
---

# The Infrastructure for Tool Usage
{: .no_toc }

*Video duration: 0:21*

A schema and a good description get a tool selected correctly; they don't make it safe or operable in production. Real tool usage needs supporting infrastructure the definition never mentions: authentication and credential scoping so a tool only acts within its intended boundary, logging and audit trails so a call can be reconstructed after the fact, rate limiting so a runaway loop can't hammer a downstream system, and sandboxing so an execution tool (code, shell commands) can't affect more than it should. None of this shows up in the tool's `input_schema` — but a tool without it is a liability regardless of how well-named it is.

Take a `run_shell_command` tool as the clearest case. Its schema might be a single, perfectly reasonable `command: string` field, and its description might be excellent — precise about what it does, when to use it, what output format to expect. None of that determines what happens when the model, following a plausible but wrong line of reasoning, generates a command that deletes more than intended or reaches outside the working directory it was meant to stay within. That's not a schema problem; a string field can't be constrained to "only safe shell commands" by any JSON Schema construct. It's solved by sandboxing — running the command in a container or restricted environment where the blast radius of *any* command, well-reasoned or not, is bounded by the execution environment itself rather than by the model's judgment in that moment.

The other three pieces of infrastructure guard different failure modes the schema is equally blind to. Credential scoping means the tool's underlying API key or service account carries only the permissions the tool actually needs — a `read_customer_record` tool authenticated with a key that also happens to grant write access is one confused or manipulated call away from an unintended write, regardless of how the tool is named or described. Logging and audit trails matter because a schema only governs what a valid call looks like going in; they're what let a team reconstruct, after the fact, which tool was called, with what arguments, by which session, and why — the same audit story the MCP governance lecture in this module treats as non-negotiable once a tool ecosystem grows past a single, easily-inspected integration. Rate limiting protects against a more mundane but equally real failure: an agent stuck in a retry loop calling the same tool dozens of times in seconds, which a perfectly valid schema does nothing to prevent.

The unifying point is that tool design and tool infrastructure are separate disciplines that happen to attach to the same object. A well-designed tool with no supporting infrastructure is still a liability; well-built infrastructure wrapped around a badly designed tool still produces selection errors. Production-grade tool usage requires both, and it's a mistake — one the exam will probe for — to treat schema quality as a proxy for operational safety.
