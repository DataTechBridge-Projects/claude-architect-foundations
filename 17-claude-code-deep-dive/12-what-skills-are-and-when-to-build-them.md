---
title: What Skills Are and When to Build Them
parent: Claude Code Deep Dive
nav_order: 12
duration: "11:00"
---

# What Skills Are and When to Build Them
{: .no_toc }

*Read duration: 11:00*

The Workflow Layer's "Skill Configuration" lecture already answered the architectural question that matters most: a Skill is packaged, invokable procedural knowledge that loads into the current agent's context only when a task calls for it, sitting in a specific niche between `CLAUDE.md` (always loaded, declarative) and a subagent (independently executed, its own context). That lecture also drew the three-way sort — `CLAUDE.md` versus Skill versus subagent — that the exam tests directly. This lecture assumes all of that as given and asks a narrower, more practically loaded question: given that a Skill is the *right kind* of mechanism for a piece of procedural knowledge, when has a team actually reached the point where building one is worth doing, versus continuing to just describe the procedure in the prompt each time it comes up?

## The Real Alternative Is Ad Hoc Prompting, Not Doing Nothing

It's easy to discuss Skills as if the choice is "build a Skill" versus "the knowledge doesn't exist anywhere." That's rarely the actual situation. Most procedures worth packaging as a Skill already exist somewhere informally — in a team member's head, in a paragraph someone retypes into the chat each time, in a stale wiki page nobody opens anymore. The real comparison a team is making is between **maintaining that knowledge as a Skill** (a reviewed, versioned artifact that loads automatically when relevant) and **continuing to re-supply it ad hoc** every time the task comes up — retyping the same instructions, or trusting that whoever runs the task that day remembers to include the right details.

Framed that way, the decision to build a Skill is an ordinary build-versus-repeat-manual-effort tradeoff, the same kind of decision that justifies writing a script instead of doing something by hand a third time. The threshold isn't "is this knowledge valuable" — almost all of it is, or it wouldn't be repeated at all. The threshold is whether the **cost of packaging it once** (writing it down clearly, structuring it, keeping it current) is lower than the **accumulated cost of re-supplying it ad hoc**, indefinitely, across everyone who ever needs it.

## Signals That a Procedure Has Crossed the Threshold

A few concrete signals tend to show up together right around the point where building a Skill starts paying for itself:

**The same instruction keeps getting retyped, nearly verbatim.** If a team member notices they're writing a close variant of the same multi-paragraph instruction for the third or fourth time — "draft a changelog entry following our format: group by feature area, link the PR, use past tense" — that's the clearest possible signal. The instruction already exists; it's just not packaged anywhere reusable yet.

**The procedure has internal structure worth getting right once.** A one-line instruction ("always run the linter before finishing") barely benefits from being packaged — it's cheap enough to just restate, and arguably belongs in `CLAUDE.md` instead, since it's small and universally relevant rather than task-specific. A procedure with several steps, judgment calls, or a specific template to follow (a compliance report with required sections and validation rules, say) benefits much more, because getting the structure right the first time and then reusing it reliably is worth considerably more than getting it approximately right from memory each time it comes up.

**The task is real but occasional, not universal.** If literally every session would benefit from a piece of knowledge, packaging it as a Skill actually costs more than it needs to — it adds a discovery step (the agent has to recognize the task matches the trigger) for something that should simply always be present. Skills earn their keep specifically for knowledge that's valuable but not universally relevant, which is the same territory the Workflow Layer identified as the Skill's home turf.

**More than one person needs it, or the same person needs it reliably over time.** A procedure only one person ever runs, once, isn't worth the investment of writing a clean, reusable version of it — just do the task. A procedure multiple people run, or one person runs repeatedly over months, justifies the fixed cost of writing it down well once, because that cost amortizes across every future invocation.

## What Ad Hoc Prompting Actually Costs, Even When It "Works"

The case for building a Skill is easy to underrate because **ad hoc prompting doesn't fail loudly — it degrades quietly**. Each time a procedure gets re-described from memory rather than pulled from a maintained artifact, there's a real chance of small drift: a step gets left out, a detail changes slightly, a newer team member describes it a bit differently than a longtime one would. None of these individual drifts look like a failure in the moment; a changelog entry still gets written, a report still gets generated. But the accumulated effect over many invocations is inconsistency that's hard to notice because there's no single artifact to compare against — every invocation was independently "close enough." A Skill converts that diffuse, hard-to-notice drift into a single maintained artifact that either reflects current best practice or doesn't, where the question "is this still accurate" has one clear place to be asked and answered, instead of being implicitly re-answered slightly differently every time someone retypes the procedure from memory.

## When Not to Build One Yet

| Build a Skill when | Hold off when |
|---|---|
| The same instruction keeps getting retyped nearly verbatim | It's only come up once so far |
| The procedure has real internal structure or judgment calls | It's simple enough to state in a single sentence |
| The task recurs, even if only occasionally | It's still actively changing week to week |
| More than one person needs it, or one person needs it repeatedly | Only one person will ever run it, once |

The mirror image of the signals above is worth stating directly, since building prematurely has its own cost. A procedure that's only come up once, that's still actively changing week to week, or that's simple enough to state in a single sentence isn't worth packaging yet — a Skill, like a `CLAUDE.md` file, is still a maintained artifact someone has to keep accurate, and writing one for a procedure that hasn't stabilized just means rewriting it repeatedly, with none of the payoff of "write once, reuse reliably" that justifies the investment in the first place. The discipline here mirrors the Workflow Layer's closing advice on maintaining Skills: it's better to wait until a procedure has genuinely stabilized and genuinely recurs than to package prematurely and end up maintaining an artifact that's really still in flux.

**Why it matters for the exam:** Where the Workflow Layer's coverage tests whether you can correctly sort a described task into `CLAUDE.md`, Skill, or subagent, this module's coverage tends to test the build-versus-not decision within the Skill category specifically — given a described recurring task, has it actually crossed the threshold that justifies packaging it, or is ad hoc prompting still the right call for now. Look for scenario questions that describe a task's frequency, stability, and audience (one person versus a team) and ask you to weigh those signals rather than just recognize "this sounds like a Skill" in the abstract.

<!-- prevnext:start -->

---

| [&larr; Previous: Creating and Structuring Custom Slash Commands](11-creating-and-structuring-custom-slash-commands) | [Next: Authoring Agent Skills &rarr;](13-authoring-agent-skills) |
|:---|---:|

<!-- prevnext:end -->
