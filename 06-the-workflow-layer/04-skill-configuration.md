---
title: Skill Configuration
parent: The Workflow Layer
nav_order: 4
duration: "15:54"
---

# Skill Configuration
{: .no_toc }

*Video duration: 15:54*

A **Skill** is a packaged, invokable procedure: a bundle of instructions — and often supporting tools, scripts, or reference files — that teaches an agent how to competently perform one well-defined kind of sub-task, on demand, without that knowledge needing to live permanently in every session's context. If `CLAUDE.md` is standing knowledge that's always loaded, a Skill is the opposite shape: knowledge that's dormant until it's relevant, then pulled into the current session precisely when it's needed.

That distinction is the whole architectural point of Skills, and it's worth sitting with before getting into mechanics. A codebase might have dozens of legitimate procedures worth teaching an agent — how to draft a release changelog, how to run a security review checklist, how to generate a data-visualization following house style, how to file a well-formed incident postmortem. If all of that lived in `CLAUDE.md`, every session would pay the context cost of loading every procedure, on every turn, regardless of whether that session ever touches changelogs or postmortems. Skills solve this by making each procedure its own self-contained package that the agent discovers and loads only when the task in front of it actually calls for it.

## What a Skill Looks Like, Architecturally

A Skill typically consists of a few pieces working together:

- **A short description or trigger.** A concise statement of what the Skill does and, critically, *when* it should be used — the signal the agent uses to decide "this task calls for that Skill" before ever loading its full contents. This is analogous to a tool's name-and-description pair, except it's describing a procedure rather than a single function call.
- **Instructions.** The actual procedural knowledge — the steps, the conventions, the judgment calls — written out the way you'd write a runbook for a competent new hire who's smart but has never done this specific task before.
- **Optionally, bundled tools or scripts.** Some Skills are pure instructions; others come with actual executable scripts, reference data, or narrow-purpose tools that only make sense in service of that one procedure. A Skill for generating a chart, for instance, might bundle a palette-validation script and reference color data alongside its instructions.
- **Optionally, reference material loaded only on demand.** Deeper documentation that the top-level instructions can point to but that doesn't need to be loaded unless the specific situation calls for that depth — a second layer of "load only when relevant" nested inside the Skill itself.

This structure should feel familiar if you've thought about tool design already: a Skill's description-and-trigger plays a role very similar to a tool's name and description, except the payload behind it is procedural instruction rather than a single function call. In both cases, the model has to correctly decide, from a short natural-language signal, whether this capability is relevant to the task at hand — which means the same discipline that applies to writing good tool descriptions (unambiguous, distinguishable from similar options, honest about scope) applies to writing good Skill triggers too.

## Discovery and Invocation

The mechanism that makes Skills work is **progressive disclosure**: the agent doesn't need the full content of every Skill in context to know that Skills exist. It typically has access to a lightweight index — names and short descriptions of the Skills available — and only loads a Skill's full instructions once it determines, from that lightweight signal, that the current task matches. This is the same "load only what's needed, when it's needed" principle that governs good context management generally, applied specifically to procedural knowledge.

This has a direct practical consequence for how Skills should be written. The trigger description is doing real work — it's the only thing evaluated on every turn, for every task, regardless of whether that task ends up using the Skill. It needs to be precise enough that the agent reliably recognizes when a task matches and doesn't reliably invoke it when a task merely sounds superficially similar. A Skill described as "helps with documents" is nearly useless as a discovery signal; a Skill described as "use this whenever generating a customer-facing changelog entry from merged PR titles" gives the agent something concrete to match against.

Once a Skill is triggered, its full instructions (and any bundled tools) become available in that session, and the agent proceeds to execute the procedure — often via its existing general-purpose tools (file read/write, code execution, etc.), guided by the Skill's specific instructions about *how* to use them for this particular task, rather than the Skill introducing some entirely separate execution mechanism.

## When a Skill Is the Right Unit

The core question this lecture is building toward is: given a piece of knowledge or a procedure, which workflow-layer mechanism should hold it — `CLAUDE.md`, a Skill, or a subagent? Skills sit in a specific, identifiable niche between the other two, and recognizing that niche is a frequently tested exam competency.

A Skill is the right unit when the knowledge is:

**Procedural rather than declarative.** `CLAUDE.md` is good at stating facts and constraints ("tests run with X," "never touch Y"). A Skill is good at teaching a *process* — a sequence of judgment calls and steps for accomplishing a specific kind of task, often with enough nuance that it wouldn't compress well into a few bullet points of standing context.

**Task-specific rather than universally relevant.** If only a fraction of sessions in a project will ever need to draft a changelog, generate a specific report format, or run a particular audit procedure, that knowledge shouldn't be paid for on every turn the way `CLAUDE.md` content is. A Skill lets it sit dormant, at zero context cost, until a task actually calls for it.

**Reusable across many invocations, by the same agent, within one session's continuous context.** This is the detail that most sharply distinguishes a Skill from a subagent, and it's worth being precise about: a Skill loads its instructions into the *current* agent's existing context and the *same* agent continues reasoning and acting with that context added. Nothing about invoking a Skill starts a new agent, opens a new context window, or makes an independent model call — it's closer to handing the current agent a runbook mid-task than to delegating the task to someone else. (The next lecture draws this distinction out fully against subagents specifically.)

**Bundled naturally with its own narrow tools.** If a procedure needs a couple of small, purpose-built scripts that only make sense in service of that one task — a formatter, a validator, a template — packaging them with the instructions that use them keeps the whole unit self-contained and easy to reason about, rather than scattering narrow-purpose tools into the project's general tool surface where they'd clutter every other task's tool selection.

## When a Skill Is the Wrong Unit

Just as important is recognizing when reaching for a Skill is the wrong call, because the exam tests both directions of this judgment.

**When the knowledge is small, stable, and needed on essentially every task.** That's the `CLAUDE.md` case, not a Skill. Wrapping "here's how we name our API routes" in a Skill adds an unnecessary discovery step for something that should simply always be present.

**When the sub-task genuinely needs its own isolated reasoning, its own context window, or independent verification separate from the calling agent's.** That's a subagent's job, not a Skill's. If a sub-task is complex enough, or noisy enough in its intermediate work, that you don't want its reasoning trace polluting the parent agent's context — or if you specifically want a fresh, independent perspective evaluating something the parent already reasoned about — a Skill can't provide that, because a Skill's content lands in the *same* context the parent is already using. Only a subagent gets a genuinely separate context and its own model call.

**When the task requires a full multi-step agentic workflow with its own decision points, tool calls, and potential need for a different model or configuration than the calling agent.** A Skill augments the current agent's capability for one step; it isn't a mechanism for spinning up a differently-configured, independently-operating unit of work.

**When the knowledge changes too fast to be worth packaging.** A Skill, like `CLAUDE.md`, is still a static artifact that someone maintains. If the "procedure" is really just "look up the current state of X," that's a retrieval problem, not a Skill.

## Configuring and Maintaining Skills Well

A few practical habits separate Skills that stay useful from ones that quietly rot or actively mislead.

**Keep the trigger and the instructions in sync as the procedure evolves.** It's easy to update a Skill's instructions after discovering a better way to do something and forget that the trigger description no longer matches the actual scope of what the Skill does. A stale trigger causes exactly the failure mode described above — missed invocations or spurious ones — even when the instructions underneath are perfectly good.

**Scope each Skill to one coherent procedure rather than letting it sprawl.** A Skill that tries to cover "release management" end to end (versioning, changelog drafting, deployment notes, rollback instructions) is really several distinct procedures wearing one trigger. Splitting it into narrower, precisely-triggered Skills usually produces more reliable invocation than one broad Skill with a vague, catch-all description, for the same reason a tool with one clear purpose beats a tool that tries to do five different things behind one ambiguous name.

**Version and review Skills the same way you'd review a `CLAUDE.md` change or a piece of code.** A Skill's instructions have leverage — they shape every future invocation of that procedure — so a bad or outdated instruction inside one is worth catching in review, not discovered the first time it produces a wrong changelog or a malformed report in production.

**Prune Skills that stop being used or stop matching current practice.** Just as a stale `CLAUDE.md` is worse than no `CLAUDE.md`, a Skill describing a procedure the team no longer follows is a trap waiting for the next task that superficially matches its trigger. Treat the Skill library as something to actively maintain, not something to only ever add to.

## A Worked Comparison

Consider a team that wants Claude to help with three different things: (1) always writing commit messages in the team's specific format, (2) generating a compliance audit report that follows a detailed, multi-section template with specific validation rules, and (3) independently reviewing a batch of pull requests overnight and posting summaries.

The commit-message convention is small, stable, and relevant to nearly every session that touches git — that's a `CLAUDE.md` entry. The compliance report is a substantial, well-defined procedure that only a minority of tasks will ever invoke, with enough internal structure (template, validation steps, maybe a formatting script) to be worth packaging — that's a Skill, invoked whenever a task matches "generate a compliance report." The overnight PR review is a genuinely independent unit of work that needs its own execution, its own context free of whatever else is happening in a session, and arguably its own model call running unsupervised — that's a subagent, not a Skill, even though from a distance "reviews PRs" and "generates reports" can sound like similarly-shaped tasks.

| Mechanism | Knowledge shape | Loaded | Own context / model call |
|---|---|---|---|
| `CLAUDE.md` | Declarative, stable, universal | Automatically, every session | No — folds into existing context |
| Skill | Procedural, occasionally needed | On demand, when a task matches the trigger | No — loads into the current agent's context |
| Subagent | Needs isolated reasoning or verification | On demand, when delegated | Yes — separate context window and model call |

**Why it matters for the exam:** Skill-configuration questions are usually framed as a three-way sort — `CLAUDE.md`, Skill, or subagent — applied to a described piece of project knowledge or a described task. The reliable way to answer them is to ask, in order: is this always relevant (favor `CLAUDE.md`), is this a well-defined, occasionally-needed procedure that can execute within the current agent's context (favor a Skill), or does this need its own isolated context and independent execution (favor a subagent). The next lecture sharpens the second-versus-third distinction specifically, since it's the one candidates most often get wrong.

<!-- prevnext:start -->

---

| [&larr; Previous: When to use CLAUDE.md vs. RAG Knowledge?](03-when-to-use-claude-md-vs-rag-knowledge) | [Next: Are Subagents Just... Skills In a Trenchcoat? &rarr;](05-are-subagents-just-skills-in-a-trenchcoat) |
|:---|---:|

<!-- prevnext:end -->
