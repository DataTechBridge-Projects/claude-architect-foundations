---
title: "Implementing Hooks: Shell Scripts, Exit Codes, and Gotchas"
parent: Claude Code Deep Dive
nav_order: 17
duration: "13:00"
---

# Implementing Hooks: Shell Scripts, Exit Codes, and Gotchas
{: .no_toc }

*Video duration: 13:00*

Knowing which lifecycle event to attach to is only half of working with hooks — the other half is the actual mechanics of writing one that behaves correctly: what a hook receives, how it communicates a decision back, and the handful of easy-to-miss details that separate a hook that works from one that silently does nothing or, worse, silently does the wrong thing.

## What a Hook Receives

A command-based hook runs as an ordinary shell script (or any executable), and Claude Code communicates with it the same way any two processes communicate over a pipe: structured data on **standard input**, as JSON. That JSON carries context about the event that triggered the hook — which event fired, the session it belongs to, the current working directory — and, for tool-related events specifically, the details of the tool call itself: which tool, and with what input. A hook attached to the before-tool-call event, for instance, receives the proposed tool name and its input arguments as part of that JSON payload, which is exactly what lets the hook actually reason about whether this specific call should be allowed, rather than reacting blindly to the fact that *some* tool call is happening.

A minimal hook script that reads this payload typically starts by capturing stdin and pulling out the fields it cares about with a JSON-processing tool:

```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')
```

Reaching for a dedicated JSON tool here rather than parsing with raw string manipulation isn't a stylistic preference — hand-rolled parsing of JSON with shell string operations is exactly the kind of fragile, error-prone approach that breaks the moment a value contains a character the ad hoc parser didn't anticipate, which is a bad property for something standing between Claude and a potentially destructive action.

## Signaling a Decision: Exit Codes

The primary channel a command hook uses to communicate its decision back is its **exit code**, and the three cases behave meaningfully differently:

**Exit code 0 — success, no objection.** The hook ran, didn't object, and execution proceeds normally. A hook can optionally also print structured JSON to stdout on a successful exit to communicate a more specific decision than a bare "proceed" — for a before-tool-call hook, this is how a hook can express something more nuanced than a plain yes, such as approving the call but flagging it, or providing additional context to the model alongside the approval.

**Exit code 2 — a blocking error.** This is the hard "no." Whatever the hook writes to standard error is fed back as the reason, and the effect depends on which event the hook was attached to: for a before-tool-call hook, exit code 2 stops the action before it ever runs at all; for an after-the-fact event like a completed tool call, the action already happened by the time the hook runs, so exit code 2 there can't retroactively undo it — it can only surface the objection back to the model as something to react to. This distinction — some events can genuinely prevent an action, others can only comment on one that already occurred — is one of the easiest details to get wrong when reasoning about what a hook attached to a given event is actually capable of.

**Any other non-zero exit code — a non-blocking error.** The hook failed to run cleanly for some reason unrelated to a deliberate objection — a bug in the script, a missing dependency, a typo — and Claude Code surfaces a notice about the failure but doesn't treat it as a decision either way; execution continues as if the hook hadn't fired an opinion at all. This distinction matters practically: a hook script with a bug that causes it to crash doesn't silently and safely allow everything through by accident, but it also doesn't accidentally block everything either — it shows up visibly as a hook error, which is the signal that something needs fixing in the hook itself, not in whatever it was supposed to be checking.

| Exit code | Meaning | Effect |
|---|---|---|
| 0 | Success, no objection | Execution proceeds; can optionally print JSON for a more nuanced decision |
| 2 | Blocking error | Before-tool-call: stops the action. After-the-fact: can only object, not undo |
| Any other non-zero | Hook failed to run cleanly | Surfaced as a hook error; treated as no opinion either way |

## A Worked Example

A hook that blocks a specific, dangerous Bash pattern, attached to the before-tool-call event, illustrates the whole mechanism end to end:

```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command // empty')

if echo "$command" | grep -qE 'rm\s+-rf\s+/(\s|$)'; then
  echo "Refusing to run a recursive delete rooted at /." >&2
  exit 2
fi

exit 0
```

Every piece of the mechanism from this lecture shows up here: reading and parsing the JSON payload from stdin, inspecting the specific field relevant to the decision (the proposed command), writing a human- and model-readable reason to stderr when objecting, and using exit code 2 specifically because this is a before-tool-call hook where blocking outright is actually possible.

## Gotchas Worth Knowing Before You Rely on One

**A hook is only as safe as the code inside it.** A hook script runs with the same real permissions as the user running Claude Code — it isn't sandboxed from the rest of the system just because it's "only a safety check." A buggy or carelessly written hook is itself a new source of risk, not a pure risk reduction, which means hook scripts deserve the same review rigor as any other piece of code with real system access — arguably more, since a hook silently failing open (exiting non-zero for the wrong reason, or matching the wrong pattern) can undermine exactly the safeguard it was meant to provide.

**Matching logic needs to be genuinely robust, not just plausible.** A hook meant to catch a dangerous shell pattern has to account for the many ways that pattern can be expressed — variable substitution, extra whitespace, a different flag ordering, the command wrapped in a subshell — the same fragility the Tool Layer warns about when discussing permission patterns that try to constrain Bash arguments. A hook whose matching logic only catches the single most obvious phrasing of a dangerous command provides much weaker protection than its author likely believes it does.

**Timeouts are real, and a slow hook is a liability.** A hook runs synchronously at its attachment point, meaning the agentic loop is genuinely paused while it executes — a hook that's slow (an unbounded network call, a large file scan) doesn't just add latency, it can time out and fail non-blocking, silently giving up its intended check exactly when it takes longest to run, which correlates unpleasantly with exactly the large, complex operations most worth checking carefully.

**Stdout from a successful hook isn't automatically silent.** Depending on configuration, what a hook prints can show up in the visible transcript rather than staying invisible plumbing, which matters for a hook meant purely for background logging — printing verbose diagnostic output on every single invocation clutters the session for no benefit, when the same information belongs in a log file instead.

**Hooks compound, and order matters.** Multiple hooks can be attached to the same event, and a badly reasoned mental model treats them as fully independent when they can interact — one hook's side effect (a file it writes, an environment variable it sets) can be visible to another hook running after it. Treating a set of hooks attached to one event as an ordered pipeline rather than a set of isolated, non-interacting checks avoids a whole category of confusing bugs.

**A hook that never fires is worse than obviously so.** The most dangerous hook failure mode isn't a hook that blocks too aggressively — that fails loudly, gets noticed, and gets fixed. It's a hook with a subtle bug in its matching logic that silently exits 0 on cases it was meant to catch, providing an illusion of a safeguard that isn't actually operating. Testing a safety-critical hook against cases it's specifically meant to catch, not just running it once and assuming it works, is the only real defense against this.

**Why it matters for the exam:** Expect questions asking what exit code a hook should return in a given scenario, and specifically testing the distinction between blocking events (where exit 2 genuinely prevents an action) and after-the-fact events (where exit 2 can only object, not undo). Also expect at least one question built around a hook that "should have" caught something but didn't — testing whether you can identify a realistic gap in matching logic or a silent-failure mode, rather than just knowing the exit-code mechanism exists in the abstract.

<!-- prevnext:start -->

---

| [&larr; Previous: Hook Lifecycle Events](16-hook-lifecycle-events) | [Next: Real-World Hook Applications &rarr;](18-real-world-hook-applications) |
|:---|---:|

<!-- prevnext:end -->
