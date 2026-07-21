---
title: "Beyond the CCAF: Additional Topics"
parent: "Extra Module: Beyond the CCA-F"
nav_order: 1
duration: "0:07"
---

# Beyond the CCAF: Additional Topics
{: .no_toc }

*Video duration: 0:07*

The CCA-F blueprint covers what you need to pass the exam: the five layers, their trade-offs, and the vocabulary to reason about them. But the five layers are a framework for making design decisions, not a complete inventory of everything a production Claude system has to get right. Once a system is live and carrying real traffic, a set of adjacent concerns shows up that the certification doesn't test directly — not because they're unimportant, but because they're operational and engineering-practice concerns layered *on top of* the architecture, rather than architecture decisions themselves.

This bonus module is a fast tour of eight of those concerns: RAG and knowledge retrieval (the concrete mechanics behind the Context Layer's "narrow the corpus down" principle), observability and telemetry (seeing what an agent actually did, not just what it returned), agent performance evaluation (proving a change helped rather than just believing it did), cost engineering (why agentic loops get expensive fast, and what to do about it), security architecture (prompt injection and the tool-surface attack surface), persistent memory systems (what an agent should remember across sessions, not just within one), CLAUDE.md/rule/prompt versioning (treating prompts as production logic), and production ops (running all of this under real load, on-call).

Each of the following lectures is intentionally short — a pointer and a mental model, not a deep dive. The goal is to give you enough vocabulary to recognize these topics when you encounter them on a real project and know roughly where to start researching, not to make you an expert in eight new subfields in the space of a few minutes each. None of this is exam-scope, so if you're purely cramming for the CCA-F, it's safe to skip this module entirely and come back to it once you're past the test and actually building.
