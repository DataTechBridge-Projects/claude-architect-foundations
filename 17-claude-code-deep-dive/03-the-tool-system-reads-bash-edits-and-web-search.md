---
title: "The Tool System: Reads, Bash, Edits, and Web Search"
parent: Claude Code Deep Dive
nav_order: 3
duration: "13:00"
---

# The Tool System: Reads, Bash, Edits, and Web Search
{: .no_toc }

*Video duration: 13:00*

The Tool Layer's "Common Claude Code Tools" lecture already walked through *why* this tool set is shaped the way it is — granularity matched per capability, errors translated rather than passed through raw, high-blast-radius capability governed outside the schema. This lecture takes a different angle on the same tool set: not why it's designed this way, but how it actually behaves in practice, session to session, and what a working knowledge of that behavior needs to include for the exam and for real use.

| Tool | What it does | Key operational detail |
|---|---|---|
| `Read` | Retrieves file contents, including some non-text artifacts | Supports offset/limit for large files; output is line-numbered |
| `Bash` | Executes shell commands | Risk is shaped jointly by permission rules, permission mode, and sandboxing |
| `Edit` | Replaces an exact string in a file | Refuses ambiguous matches; requires the file to have been read first in-session |
| `WebFetch` | Retrieves a specific, already-known URL | Distills content toward a stated extraction goal |
| `WebSearch` | Runs a query against a search engine | Returns candidate results for discovery, not full page content |

## Read: more than "open a file"

`Read` retrieves file contents, but its practical behavior has a few details worth knowing precisely. It supports an offset and a limit, so Claude can pull a specific slice of a large file rather than the whole thing — important for files too large to comfortably fit in context, where reading the entire file would burn a disproportionate share of the context budget on content that isn't relevant to the current task. Output comes back with line numbers prefixed, which matters mechanically: it's what lets a subsequent `Edit` call reference an exact location, and it's why line-numbered output should never itself be mistaken for file content when reasoning about what a file actually contains.

`Read` can also handle non-text content in some configurations — images, for instance — which is a reminder that "read a file" in an agentic coding harness isn't purely a text operation; it's "bring the relevant artifact into context in whatever form the model can reason about it." The practical implication is that Claude Code's file-reading capability isn't limited to source code the way a narrower "read source file" tool might be — it's closer to a general-purpose "look at this artifact" capability, scoped by what the underlying model can actually interpret.

## Bash: what actually runs, and what gets stopped

`Bash` executes shell commands, and the operationally important thing to internalize is that its risk isn't uniform across all invocations — it's shaped by three layers working together: **permission rules** (what's allow-listed, deny-listed, or requires a prompt), **permission mode** (covered in full in the next lecture — how aggressively those prompts get skipped), and, where configured, **sandboxing** (OS-level restriction on what the process can actually reach on disk or over the network, independent of what Claude Code's own permission logic decides). These three are complementary, not redundant: a permission rule stops Claude from *attempting* a disallowed command; a sandbox stops a command from *succeeding* even if something upstream — a prompt injection from a file Claude read, for instance — talked Claude into attempting it anyway. Relying on permission rules alone assumes the model's decision-making is the only thing standing between a request and a destructive command; a sandbox adds a layer that holds even if that assumption fails.

Claude Code also recognizes a fixed set of Bash invocations as read-only — directory listings, `git status`, `grep`, and similar commands that can't mutate anything — and runs these without a permission prompt in effectively every mode, since prompting for something that structurally can't cause harm just adds friction without adding safety. Everything else defaults to requiring approval unless a rule or mode says otherwise. Practically, this means the single most common friction point in early Claude Code use is discovering which everyday commands fall on which side of that line, and building up allow-list rules (or accepting a looser permission mode) for the commands a given project genuinely needs run repeatedly.

A second practical detail: compound shell commands (`a && b`, `a | b`) are evaluated per sub-command, not as one opaque string. A permission rule written for one command doesn't implicitly grant a chained command that includes it, and a properly implemented permission layer parses shell operators rather than doing a naive substring match — a chained `safe-command && rm -rf /` should not be silently approved just because `safe-command` alone is allow-listed. Understanding this is what separates a permission configuration that only *looks* safe from one that actually is.

## Edit: exact match, read-first, and why it fails loudly

`Edit`'s exact-match requirement — replacing a specific old string with a new one, only where that old string appears — has a very concrete practical consequence: if the target text appears more than once in a file, `Edit` refuses to guess which occurrence you meant and returns an error instead, forcing either more surrounding context in the match or an explicit choice about which instance to change. This is a deliberate refusal to be "helpful" in a way that could silently do the wrong thing. The same logic underlies the read-before-edit requirement most configurations enforce: Claude Code tracks whether a file has actually been read in the current session before it will let that file be edited, precisely so an edit is never made against a stale, remembered version of a file that may have changed since the model last looked at it (from another tool call, a hook, or an external process).

Practically, this means a session that edits the same file repeatedly benefits from re-reading it after significant external changes (a formatter running, a hook modifying it, a human editing it in parallel), and it's why "Claude tried to edit a file it hadn't read in this turn" errors, when they occur, are a safety feature working as intended rather than a bug to route around.

## WebFetch and WebSearch: two different jobs

Claude Code exposes web access as two distinct tools rather than one, and the distinction matters operationally. `WebFetch` retrieves and processes the content of a *specific, known URL* — you already know where you want to look, and the tool's job is to bring that page's content into context, generally distilled toward whatever extraction goal was specified rather than dumped in raw. `WebSearch` runs a *query* against a search engine and returns a set of results — you don't yet know the specific URL, and the tool's job is discovery: finding candidate pages worth fetching next.

The practical workflow this implies is usually sequential — search to find the right page, then fetch it for detail — and conflating the two (expecting `WebFetch` to discover something, or expecting `WebSearch` to return full page content) is a common source of confused tool selection in less experienced use. Both tools inherit the same trust-boundary caution the Tool Layer raised about external content generally: text retrieved from the open internet is data for the model to reason about, not an instruction to follow, and a well-configured Claude Code session applies domain allow-lists or deny-lists to constrain which hosts either tool can reach at all — directly reusing the permission-rule mechanism that governs every other tool.

## Putting it together: a realistic sequence

A representative task — "fix the failing test in `auth.spec.ts` and confirm the fix" — exercises this tool system in a characteristic order: `Read` (or a search tool) to find and inspect the failing test and the code it exercises, `Bash` to actually run the test suite and see the failure, `Read` again on whatever source file needs changing, `Edit` to make the targeted fix, `Bash` again to re-run the tests and verify, and possibly `WebFetch` or `WebSearch` along the way if the fix requires checking a library's current API against its documentation. Nothing about this sequence is scripted in advance — it emerges from the model reasoning about what it needs next, with the tool system supplying exactly the affordances (and the exactly the guardrails) needed to carry it out safely.

**Why it matters for the exam:** Where the Tool Layer's coverage of these same tools tests *design rationale* ("why is granularity split this way"), this module's coverage tests *operational literacy* — what a specific tool actually does with a given input, why a specific action gets blocked or auto-approved, and how permission rules, modes, and sandboxing combine. Expect scenario questions that give you a concrete command or edit and ask what happens next, not just what principle it illustrates.

<!-- prevnext:start -->

---

| [&larr; Previous: What Claude Code Is and How It Works](02-what-claude-code-is-and-how-it-works) | [Next: Starting a Project: Onboarding Claude to a Codebase &rarr;](04-starting-a-project-onboarding-claude-to-a-codebase) |
|:---|---:|

<!-- prevnext:end -->
