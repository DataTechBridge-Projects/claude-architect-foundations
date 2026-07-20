---
title: Lab: AI Workforce Allocation Simulator
parent: The Agentic Layer
nav_order: 16
duration: "2:07"
---

# Lab: AI Workforce Allocation Simulator
{: .no_toc }

*Video duration: 2:07*

This lab is a hands-on exercise in the tradeoffs this module has been building toward, framed as a resource-allocation problem rather than a quiz. You're given a small backlog of tasks — some independent, some sequentially dependent, varying in complexity, acceptable latency, and how costly a wrong answer would be — and a constrained budget of "agent capacity": a limited number of concurrent agents you can run, a per-task cost ceiling, and an overall latency target for the batch.

Your job is to decide, for each task, how it should be handled: run it inline within a single orchestrating agent, dispatch it as a fresh-context subagent, batch it in parallel alongside independent siblings, or run it sequentially behind a dependency. For each choice, you also decide how much safeguarding it warrants — whether it needs a checkpoint or human approval gate before completing, based on the reversibility and blast radius of what it does — and how tightly to scope its tool access.

The simulator scores your allocation against three axes: total cost, total wall-clock time to complete the batch, and a risk score that penalizes high-blast-radius tasks left ungated or overscoped. There is no single correct allocation — the exercise is designed so that optimizing purely for speed (maximal parallel dispatch) or purely for safety (a checkpoint on everything) both produce a worse overall score than an allocation that matches each task's actual uncertainty, dependency structure, and stakes, the same judgment call this module has been asking you to make throughout.

Run the simulation a few times with different priority weightings (favor cost, favor speed, favor safety) and notice how your allocation strategy — not just the numbers — has to change. That shift in reasoning is the actual skill being tested, both here and on the exam.
