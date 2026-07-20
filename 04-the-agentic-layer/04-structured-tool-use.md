---
title: Structured Tool Use
parent: The Agentic Layer
nav_order: 4
duration: "14:32"
---

# Structured Tool Use
{: .no_toc }

*Video duration: 14:32*

Step 3 of the orchestration loop — "branch on the response" — only works reliably because tool-use requests aren't free-form text. They're **structured**: a specific tool name plus a set of arguments that conform to a schema you defined in advance, almost always JSON Schema. This lecture is about why that structure is the load-bearing design choice that makes the whole loop dependable, and what breaks when you don't have it.

## The alternative: asking the model to describe actions in prose

Imagine, instead, a model that expresses its intent to act entirely in natural language: "I should read the file config.yaml to check the current settings." A surrounding system that wanted to act on this would have to parse that sentence, figure out that it's expressing an intent to call a `read_file`-shaped action, extract `config.yaml` as the argument, and hope the model phrased things consistently enough across thousands of similar cases to make that parsing reliable. This is not a hypothetical — it's roughly how early LLM agent frameworks worked before native tool-calling APIs existed, using prompt conventions ("Action: read_file(config.yaml)") and regular expressions or secondary LLM calls to extract structure from prose after the fact.

The problems with this approach are architectural, not cosmetic:

- **Parsing is probabilistic on top of probabilistic.** The model's underlying generation is already a probability distribution over tokens; extracting structure from its prose output with regex or a second model call adds a second layer of unreliability on top of the first. A phrasing variation ("I'll go ahead and check config.yaml" vs. "Let me read config.yaml") that's semantically identical to a human can silently break a brittle parser.
- **Arguments are ambiguous.** Free text doesn't distinguish a filename from a description of a filename, doesn't enforce types (is `"5"` a string or a number?), and doesn't handle optional versus required parameters in any consistent, machine-checkable way.
- **There's no validation boundary.** With prose, there's no natural point at which the system can say "this request is malformed, reject it before executing anything" — the ambiguity is only discovered when execution fails, or worse, silently succeeds with the wrong arguments.
- **Composability collapses.** If you have a dozen available actions, prose-based intent detection has to disambiguate among all of them from unstructured language, which gets harder — not easier — as the tool set grows.

## What structured tool use actually is

**Structured tool use** (also called **function calling**) means the developer defines, for each available tool, a name, a natural-language description of what it does and when the model should use it, and a JSON Schema describing its parameters — types, which are required, permitted value ranges or enums, and so on. These definitions are sent alongside the prompt on every model call. When the model decides an action is warranted, it doesn't describe that action in prose — it emits a response object with a distinct content type (a tool-use block) naming the specific tool and providing arguments that conform to the declared schema. The Claude API represents this as structured JSON in the response, separate from any accompanying text, specifically so the harness can detect it programmatically rather than by pattern-matching prose.

A concrete shape, using a hypothetical `search_issues` tool:

```json
{
  "name": "search_issues",
  "description": "Search the issue tracker for issues matching a query. Use this before creating a new issue to check for duplicates.",
  "input_schema": {
    "type": "object",
    "properties": {
      "query": { "type": "string", "description": "Search terms" },
      "status": { "type": "string", "enum": ["open", "closed", "all"] },
      "limit": { "type": "integer", "minimum": 1, "maximum": 50 }
    },
    "required": ["query"]
  }
}
```

When the model calls this tool, the response contains a tool-use block like `{"type": "tool_use", "name": "search_issues", "input": {"query": "login timeout", "status": "open", "limit": 10}}`. That's not prose to be interpreted — it's data the harness can validate against the schema, route to the right handler function, and execute directly, with `query` guaranteed present and a string, `status` guaranteed to be one of exactly three values if present at all, and `limit` guaranteed to be an integer in range if present.

## Why structure improves reliability

The core reliability gain is that structure moves error detection earlier and makes it deterministic. A malformed tool call — a missing required field, a value outside an enum, a string where a number was expected — can be caught by schema validation before any real-world action is attempted, and the harness can return a clear, structured error to the model ("missing required field: query") rather than attempting execution with garbage input and failing in some downstream, harder-to-diagnose way. This is the same principle behind strongly typed interfaces in traditional software: pushing a class of bugs from runtime behavior into a validation step you can reason about and test.

Structure also reduces the space of things that can go wrong in the model's own generation. Constrained to a known schema, the model is choosing among a bounded set of well-defined tools and filling in well-defined slots, rather than freely composing sentences that then have to be reverse-engineered into structure. In practice this measurably improves the rate at which tool calls are both syntactically valid and semantically sensible, particularly as the number of available tools grows — exactly the composability problem prose-based approaches struggle with.

## Why structure improves parseability

Parseability is almost definitional here: JSON Schema-defined arguments are machine-readable by construction, using the same parsing and validation code you'd use for any other structured API payload — no custom prose-extraction logic, no regular expressions tuned to catch phrasing variants, no secondary model call just to figure out what the first model call meant. This has a direct engineering consequence: tool implementations can be ordinary typed functions. A `search_issues` handler can declare `query: str, status: Literal["open","closed","all"] = "all", limit: int = 20` in whatever language the harness is written in, and the mapping from the model's structured output to that function signature is direct and mechanical, not interpretive.

## Why structure improves composability

Composability is where structured tool use compounds in value as systems scale. Because every tool exposes the same shape — name, description, schema — tools can be defined independently, by different teams or even different organizations (this is exactly the premise of the Model Context Protocol, covered in the Tool layer), and combined into a single tool set for a given agent without any of them needing to know about each other. The model disambiguates among them using the descriptions and schemas alone. Contrast this with a prose-based system, where adding a new action means updating whatever bespoke parsing logic detects intent from language, and where two actions with similar natural-language descriptions ("check the weather" vs. "check the forecast") are much harder to keep unambiguous than two tools with distinct names and distinct required parameters.

This composability is also what makes multi-agent and subagent architectures (later in this module) practical. An orchestrator can present a delegation action — "invoke the research subagent with this task description" — as just another structured tool, indistinguishable in kind from "read this file" or "run this query." The subagent, in turn, has its own tool set. Structure is what lets these compose into an agent hierarchy of arbitrary depth without a fundamentally different calling convention at each level.

## Structured tool use enables parallel and multi-call turns

One consequence of structure that's easy to overlook: because each tool-use request is a discrete, self-contained object with its own ID, a single model response can contain more than one of them at once. The model might, in one turn, request three independent tool calls — say, reading three different files it already knows it needs — and the harness can execute all three (potentially concurrently) and return three separate tool results, each matched back to its originating request by ID. This is difficult to do reliably with prose-based intent detection, where disentangling "the model wants to do these three things" from a single block of free text is a much harder parsing problem than reading three discrete, independently-schema'd objects out of a response. Multi-call turns are also part of why granularity choices (small, focused tools versus fewer, broader ones) matter for the latency discussion in the orchestration loop lecture: a model that can request several small, structured actions in one turn recovers some of the efficiency that granularity would otherwise cost.

Structure also makes error recovery mid-loop tractable in a way prose can't. When a tool call fails — the schema validated fine, but the underlying action errored (a file didn't exist, an API timed out) — the harness returns a structured error as the tool result, and the model can reason about that specific, well-defined failure and decide whether to retry with different arguments, try a different tool, or surface the failure to the user. Because the error is itself structured (an error type, a message, sometimes a code) rather than a vague prose failure description, the model has a far better basis for choosing a sensible next action than it would parsing an unstructured stack trace or log dump. This connects directly to the Tool layer's treatment of error response design: a well-structured tool doesn't just define its success case cleanly, it defines its failure cases with the same rigor, precisely because the model's recovery behavior in the next loop iteration depends on it.

## How this shows up in Claude Code and the Claude API

At the API level, this is the `tools` parameter on a Messages API request: an array of tool definitions, each with `name`, `description`, and `input_schema`, exactly as sketched above. The model's response includes `tool_use` content blocks when it elects to call one, and your application sends a `tool_result` content block back in the next user-role message, referencing the tool call by its ID so the model can correctly associate results with requests even when multiple tools were called in the same turn.

Claude Code is a concrete, fully built-out instance of this pattern: `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, and the rest are all tool definitions with real schemas, registered with the harness, executed by real handler code, with results returned as structured tool results. When you write a CLAUDE.md instruction like "always run tests after editing," you are shaping *when* the model chooses to call the `Bash` tool — you are not, and cannot, redefine the schema of the tool itself, which is fixed by the harness. That separation — prompts shape the model's judgment about which structured actions to take and when; schemas fix what those actions can structurally look like — is a distinction worth holding onto, because it recurs across the entire Prompt and Tool layers.

## Why it matters for the exam

Expect questions that ask you to identify why a described system is fragile — commonly, a system that relies on parsing the model's free-text output to detect intended actions, rather than using native structured tool calling. You should be able to state precisely what breaks in that design (ambiguous parsing, no validation boundary, poor composability as tools are added) and what structured tool use fixes about each. You should also be comfortable with the basic anatomy of a tool definition — name, description, JSON Schema for parameters — since exam scenarios will describe tool definitions and ask you to reason about how the model would use them.
