---
title: "Lab: Prompt Reliability Simulator"
parent: The Prompt Layer
nav_order: 14
duration: "1:42"
---

# Lab: Prompt Reliability Simulator
{: .no_toc }

*Video duration: 1:42*

This lab is a hands-on companion to the whole module, built around a single deliberately fragile starting prompt: a ticket-triage prompt that classifies incoming support tickets by urgency and routes them to a team. Run as given, it performs reasonably on a handful of clean, typical tickets — and then falls apart against a bundled test set of twenty-five varied inputs, including terse one-line tickets, tickets in a mix of registers, a few that mention dollar amounts without being urgent, and a few genuinely urgent tickets with no financial language at all.

Your task is iterative: run the starting prompt against the full test set, log where it misclassifies, and diagnose *why* — is it an ambiguous success criterion, a few-shot example that taught the wrong lesson through an incidental correlation, a missing structured-output constraint letting stray text through, or a missing self-check that would have caught an obviously wrong classification before it shipped? Apply exactly one fix at a time, re-run the full test set, and record whether accuracy improved, stayed flat, or — instructively — got worse on a different slice of inputs than the one you were targeting.

By the end, you'll have a short changelog mapping specific prompt edits to specific reliability outcomes across a fixed test set — which is exactly the discipline a production prompt owner exercises every time a prompt changes, and exactly the kind of before/after reasoning the exam expects you to be able to walk through.

## What a Typical Fix Cycle Looks Like

A representative pass through the lab might start by noticing that eleven of the twenty-five test tickets are misrouted, and that nine of those eleven have one thing in common: they're short, one-line tickets, while every few-shot example in the starting prompt runs two to three sentences. That's the exact failure mode covered earlier in this module — narrow example coverage teaching the model a pattern that's a subset of the real input range. The first fix, then, isn't rewriting the classification logic; it's swapping one of the existing few-shot examples for a terse one-liner and adding a second short example on the non-urgent side, so length stops being an accidental signal. Re-running the full test set after that single change typically recovers most of those nine misroutes — but not all of them, and watching exactly which ones remain broken after an otherwise-successful fix is where the real diagnostic skill in this lab lives.

## Why the Regressions Matter as Much as the Fixes

The instruction to record cases where accuracy "got worse on a different slice" isn't a throwaway detail — it's the core discipline the lab is built to teach. Prompt changes are rarely purely additive; tightening a success criterion to fix one failure mode can easily introduce a new one elsewhere, especially when a fix is validated only against the specific failing case that motivated it rather than the full test set. This is exactly why the lab insists on re-running all twenty-five tickets after every single change, one at a time, rather than batching several plausible fixes together and checking only the cases you were targeting — batching changes makes it impossible to attribute a regression to the specific edit that caused it. That discipline mirrors the independent-review theme explored elsewhere in this module: a fix that looks obviously correct in isolation still needs to be checked against a standard broader than the case that inspired it, the same way a review pass needs a standard broader than the drafting pass's own reasoning.
