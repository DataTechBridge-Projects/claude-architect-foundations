---
title: Common Claude Code Tools
parent: The Tool Layer
nav_order: 10
duration: "16:45"
---

# Common Claude Code Tools
{: .no_toc }

*Video duration: 16:45*

Everything in this module so far has been principle: how to name a tool, how to write its description, how to design its errors, how to think about MCP and governance. This lecture grounds all of it in a system you likely already have direct experience with — Claude Code's own built-in tool set — and reads it as a worked example, asking of each tool the same question this module has been asking throughout: why is it shaped the way it is, and what would go wrong if it weren't?

## File tools: Read, Write, Edit

Claude Code's file-manipulation surface is split into three distinct tools rather than one general-purpose "file operation" tool, and that split is itself a granularity decision worth examining. `Read` retrieves a file's contents (optionally a line range, for large files). `Write` creates a file or overwrites one entirely. `Edit` performs a targeted, exact-match string replacement within an existing file, changing only the specific text that changed rather than rewriting the whole file.

The case for splitting these into three tools rather than one `file_operation` tool with a `mode` parameter is a direct application of the granularity principle from earlier in this module: reading, fully overwriting, and surgically editing are operations with meaningfully different risk profiles and meaningfully different argument shapes. Collapsing them into one tool would mean a single schema trying to accommodate "sometimes you need a line range, sometimes you need entire new content, sometimes you need an old-string/new-string pair," which is exactly the tangled, cross-field-dependent parameter space the earlier lecture warned produces subtly wrong calls. Keeping them separate means each tool's schema is simple and each tool's description can be precise about exactly when to reach for it — and, notably, `Edit` requiring an exact match of the existing text (rather than a fuzzy line-number-based patch) is itself an error-design choice: it fails loudly and specifically ("old_string not found" or "old_string appears multiple times, not unique") rather than silently editing the wrong occurrence, which is exactly the "fail specifically rather than fail silently" principle from the error-response lecture.

It's also worth noting the deliberate coupling between `Read` and `Edit`: Claude Code requires a file to have been read before it can be edited in most configurations. That's a workflow-safeguarding decision as much as a tool-design one — it prevents a blind edit against content the model hasn't actually verified, reducing the odds of a confidently wrong edit made against stale assumptions about a file's current contents. It's a concrete instance of a tool set enforcing a sane sequence of operations structurally, rather than relying on a prompt instruction ("always read before editing") that the model might not reliably follow on its own.

## Bash: broad capability, narrow trust boundary

`Bash` is the widest-reaching tool in Claude Code's default set — it can run arbitrary shell commands, which means, in principle, it can do almost anything the file tools, search tools, and more can do, and a great deal they can't. This makes it the clearest illustration in the whole tool set of the coarse-versus-fine granularity tradeoff discussed earlier in this module, applied at the extreme coarse end: rather than defining a narrow tool for every conceivable shell operation (a `git_status` tool, a `run_tests` tool, an `install_package` tool), Claude Code exposes one broad-capability tool and relies on the model's judgment, description-level guidance, and surrounding safeguards to use it well.

That design only works because of infrastructure that sits around the tool rather than inside its schema: permission prompts before consequential commands run (in interactive use), configurable allow-lists and deny-lists that scope what commands are permitted without a human in the loop, sandboxing that can constrain what a command is actually able to touch, and — in the underlying harness — timeouts so a hung command doesn't stall the loop indefinitely. This is the tool-usage infrastructure aside from earlier in this module made concrete: `Bash`'s schema alone (a `command` string and not much else) says almost nothing about safety; the actual safety comes from the surrounding permission system, which is exactly the point that aside was making — a schema defines validity, not governance.

`Bash`'s error responses are also a useful case study, because a failed shell command's natural output — stderr, a non-zero exit code — is close to the "raw stack trace" problem the error-design lecture warned against, and Claude Code's handling of it (returning captured stdout and stderr together, with clear indication of the exit code and whether the command was truncated or timed out) is a deliberate translation layer, not a raw passthrough of whatever the underlying shell happened to emit.

## Search tools: Grep and Glob

`Grep` (content search across files, built on ripgrep) and `Glob` (filename pattern matching) are a clean illustration of the "narrow, well-differentiated tools beat one overloaded tool" side of the granularity tradeoff, in deliberate contrast to `Bash`'s "one broad tool" approach for a different kind of capability. Search is common enough, and the two dimensions of it (matching by content versus matching by filename pattern) are distinct enough, that splitting them into two sharply-scoped tools produces less ambiguity than either forcing all search through `Bash` (technically possible, since `grep` and `find` are shell commands, but far more error-prone for the model to construct correctly as raw shell invocations every time) or collapsing content and filename search into one tool with a mode flag.

This is worth sitting with as a direct rebuttal to the idea that "fewer tools is always better" — the lesson from `Bash` isn't "prefer broad tools," it's "match granularity to how distinct and how frequently-needed the underlying operations are." Shell execution is broad because enumerating every possible shell operation as a separate tool would be absurd — the space is unbounded. Search is narrow and split because the space is small, well-defined, and the two operations within it (content match, name match) are genuinely different enough in their parameters and their typical use that one schema trying to cover both would be worse than two clean ones. The existence of both patterns side by side in the same tool set is itself the lesson: granularity is a judgment call made per capability, not a global policy.

Note also that `Grep` and `Glob` exist specifically so the model doesn't have to reach for `Bash` and hand-construct a `grep -rn` or `find` invocation with correct flags every time it needs to search — which connects back to the "multiple pathways" aside: the model *could* accomplish search through `Bash` alone, but a dedicated tool with a tighter, purpose-built schema produces more reliable calls than routing a common operation through the maximally general escape hatch every time.

## WebFetch: bringing the outside world into context

`WebFetch` retrieves and processes the content of a URL, which makes it structurally different from the file and search tools in one important way: everything the other tools return originates from a controlled, local environment (the filesystem, the shell), while `WebFetch`'s results originate from the open internet. This is exactly the trust boundary raised in the MCP governance discussion — a tool result is data, not instruction, and that principle applies with particular force here, since fetched web content could contain adversarial text specifically crafted to look like an instruction to whatever model processes it. `WebFetch`'s design — typically summarizing or processing fetched content with a specific extraction goal, rather than dumping an entire raw page verbatim into context — is partly a context-management concern (raw HTML is enormous and mostly noise) and partly a security-adjacent one: constraining what comes back reduces the surface for injected content to reach the orchestrating model's own instructions unfiltered.

## Task: delegation as a tool call

The `Task` tool (spawning a subagent) is the clearest example in Claude Code's own tool set of a point made in the Agentic Layer's structured-tool-use lecture: delegation to a subagent is, mechanically, just another structured tool call — a name, an input schema (typically a task description and sometimes a target subagent type), and a result that comes back as a tool result once the subagent's own independent loop completes. There's no separate calling convention for "spawn an agent" versus "read a file"; both are the same kind of structured action from the orchestrator's point of view, which is exactly what makes agent hierarchies of arbitrary depth possible without inventing a new mechanism at each level.

Framed through this module's vocabulary rather than the Agentic Layer's, `Task` is also a good example of a tool whose description carries unusually high stakes for correct selection, because the consequences of over- or under-using it are asymmetric and expensive: calling it when a task doesn't warrant a fresh, isolated context wastes the overhead of a whole separate agent loop for something that could have been done inline; not calling it when a task genuinely would benefit from context isolation lets exploratory noise pollute the orchestrator's own context instead. Its description in a well-designed harness has to communicate that judgment call clearly, since — unlike `Grep` versus `Glob`, which differ in an easily stated dimension (content versus filename) — `Task` versus "just do it inline" is a genuinely harder distinction to convey precisely in a short description, and it's one of the more sophisticated pieces of tool-selection guidance in the whole default tool set.

| Tool(s) | Granularity approach | Key principle illustrated |
|---|---|---|
| `Read`, `Write`, `Edit` | Split three ways by risk and argument shape | Narrow tools for operations with different risk profiles |
| `Bash` | One broad tool | Safety lives in surrounding infrastructure, not the schema |
| `Grep`, `Glob` | Split two ways by search dimension | Narrow, well-differentiated tools beat one overloaded one |
| `WebFetch` | One tool, tightly scoped output | Content from the open internet is untrusted data, not instruction |
| `Task` | One tool, high-stakes description | Delegation is just another structured tool call |

## Reading the whole set as a system

Looking at the tool set as a whole rather than tool by tool, a few patterns recur that map directly onto this module's principles:

- **Granularity is matched to the operation, not applied uniformly.** File operations are split three ways; search is split two ways; shell execution is one broad tool; delegation is one tool with high-stakes description work behind it. There's no single granularity policy — each capability got the granularity that fit its own shape.
- **Naming is consistent and verb-forward.** `Read`, `Write`, `Edit`, `Grep`, `Glob`, `Task` — short, action-oriented, and none of them ambiguous about their core verb, even before reading a full description.
- **High-blast-radius capability is governed outside the schema.** `Bash`'s permission system, not its input schema, is what makes it safe enough to expose at all — a direct instance of the "infrastructure beyond the definition" aside.
- **Errors are translated, not passed through raw.** Failed edits, failed commands, and failed fetches all come back through a designed layer rather than as raw exceptions, consistent with the error-response lecture's core argument.
- **Tool results are treated as data a reasoning system consumes, not documentation for a human.** Search results, file contents, command output, and fetched web content are all shaped with the *next model call* as the intended reader, which is the thread running through this entire module.

None of this is coincidental — it's the same discipline this module has been teaching, already applied, in a tool set you can inspect directly by using Claude Code itself. That's a genuinely useful study habit going into the exam: when a question describes an unfamiliar tool-design scenario, ask which of these already-familiar patterns it most resembles, and reason from there.

**Why it matters for the exam:** Expect questions that describe one of Claude Code's built-in tools, or a hypothetical tool clearly modeled on one, and ask you to identify which design principle from this module it illustrates — granularity matching, infrastructure-gated safety, translated error responses, or trust boundaries around external content. Being able to name the concrete, familiar example (not just the abstract principle) is often the fastest way to reason through an unfamiliar scenario question under time pressure.

<!-- prevnext:start -->

---

| [&larr; Previous: The Infrastructure for Tool Usage](09-the-infrastructure-for-tool-usage) | [Next: Different Workflows, Similar Principles &rarr;](11-different-workflows-similar-principles) |
|:---|---:|

<!-- prevnext:end -->
