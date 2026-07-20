---
title: Stress Testing - Context Architecture
parent: Extra Module: Solution Architecture Patterns - Banking Compliance
nav_order: 3
duration: "5:51"
---

# Stress Testing - Context Architecture
{: .no_toc }

*Video duration: 5:51*

Every stage of the stress-testing workflow described in the previous two lectures depends on getting one thing right first: making sure Claude sees the correct version of a set of documents and data that all change, on different schedules, every cycle. Context architecture in this domain is less about volume — the individual documents involved aren't huge by RAG standards — and more about discipline around *which version* of each input is in play at any moment.

## The Three Core Context Inputs

Three categories of information have to be assembled into context at each stage, and each has its own volatility profile. **Scenario definitions** are the regulator's published macroeconomic assumptions for the current cycle — unemployment paths, GDP trajectories, interest rate curves, and the dozen or so other variables that define the baseline, adverse, and severely adverse scenarios. These are published once per cycle and must not be confused with any prior cycle's scenario, even though the variable names and general shape look similar year over year. **Portfolio composition** is the bank's own current book — loan balances, exposures, maturities, collateral characteristics — segmented by the portfolio categories the models operate on. This changes continuously in the real world but has to be pinned to a specific, timestamped snapshot for the purposes of a given stress-test run, so that a number can be reproduced later against the exact portfolio state that produced it. **Prior-cycle results** are the bank's own submitted stress-test outputs from previous cycles, which the narrative-drafting stage needs in context specifically to explain *why this cycle's results differ* — a question regulators reliably ask, and one the system should be able to answer proactively rather than reactively.

## Why Versioning Is Non-Negotiable

The failure mode unique to this domain is subtle: a scenario document, a methodology memo, or a portfolio snapshot from the wrong cycle can be individually well-formed and internally consistent, which means an error caused by mixing cycle versions won't look obviously wrong to either the model or a distracted human reviewer. Last year's severely adverse scenario and this year's severely adverse scenario share the same structure and similar magnitude; a system that silently pulls the wrong one produces plausible-looking numbers built on the wrong premise. Guarding against this means every document entering context carries an explicit, machine-checked cycle identifier — not just a filename convention a human is trusted to get right, but a metadata tag the retrieval and assembly layer validates before anything is included in a prompt. A scenario document without a verified current-cycle tag should be treated as a retrieval failure, not silently included with a caveat.

This is also why prior-cycle results are handled differently from current-cycle inputs: they're deliberately included as **labeled historical context** — explicitly marked as "prior cycle, for comparison only" — rather than left to blend in among current-cycle data where a model (or reviewer) might mistake them for this year's figures. The context assembly layer's job isn't just fetching the right documents; it's making the cycle boundary visually and structurally unmistakable in how those documents are presented.

## Portfolio Snapshots and Reproducibility

Because portfolio composition changes daily but a given stress-test run has to be reproducible months later during an examination, the system never queries "current portfolio" live at generation time. Instead, each run references an immutable, timestamped snapshot, and that snapshot's identifier travels with every downstream artifact — model outputs, narrative drafts, review records — so that reconstructing exactly what the system saw at generation time is a lookup, not a reconstruction exercise. This is a context-management discipline as much as a data-engineering one: the architecture treats "what portfolio state did this run see" as a first-class piece of provenance metadata, not an implicit assumption.

## Architecture Notes

This is the **Context** layer's data-provenance concern in its sharpest form: two documents can be individually well-formed and still be catastrophically wrong together if they come from different regulatory cycles, so provenance tracking has to be enforced structurally rather than left to convention. The discipline of pinning immutable portfolio snapshots also supports the reproducibility requirements the **Workflow** layer's audit gates depend on — a run can only be defended later if exactly what it saw is recoverable.
