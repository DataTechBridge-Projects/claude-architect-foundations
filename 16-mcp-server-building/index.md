---
title: MCP Server Building
nav_order: 17
has_children: true
permalink: /16-mcp-server-building/
---

# MCP Server Building

Goes beneath the conceptual MCP architecture covered in the Tool Layer into the mechanics of actually building servers and clients: the three MCP primitives (tools, resources, prompts), writing a real server in Python, choosing between STDIO and remote transports, and less common but exam-relevant mechanisms like sampling, notifications, roots, and progress reporting. Closes by contrasting all of this against Claude's built-in tools and a decision framework for when MCP is worth reaching for at all.

**Official docs:** [Model Context Protocol](https://modelcontextprotocol.io/) · [MCP connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector)

## Lectures

| # | Lecture | Duration |
|---|---------|----------|
| 1 | [Module Intro](01-module-intro) | 2:30 |
| 2 | [Tools, Resources & Prompts: The Three MCP Primitives](02-tools-resources-and-prompts-the-three-mcp-primitives) | 15:00 |
| 3 | [Building an MCP Server in Python](03-building-an-mcp-server-in-python) | 20:00 |
| 4 | [Defining and Exposing Tools via MCP](04-defining-and-exposing-tools-via-mcp) | 14:00 |
| 5 | [Defining and Exposing Resources](05-defining-and-exposing-resources) | 12:00 |
| 6 | [MCP Prompt Templates](06-mcp-prompt-templates) | 10:00 |
| 7 | [Implementing MCP Clients](07-implementing-mcp-clients) | 13:00 |
| 8 | [STDIO Transport: Local Servers and Process Communication](08-stdio-transport-local-servers-and-process-communication) | 11:00 |
| 9 | [StreamableHTTP Transport: Remote Servers and Production Deployment](09-streamablehttp-transport-remote-servers-and-production-deplo) | 13:00 |
| 10 | [Letting Servers Make LLM Calls Through the Client](10-letting-servers-make-llm-calls-through-the-client) | 12:00 |
| 11 | [Notifications, Roots, and Progress Reporting](11-notifications-roots-and-progress-reporting) | 11:00 |
| 12 | [The Built-in Toolset: What Ships with Claude](12-the-built-in-toolset-what-ships-with-claude) | 14:00 |
| 13 | [Choosing Built-in vs Custom Tools](13-choosing-built-in-vs-custom-tools) | 10:00 |
| 14 | [Module Outro](14-module-outro) | 2:00 |
| 15 | [Key Distinctions - MCP Server Building](15-key-distinctions-mcp-server-building) | 4:00 |
| 16 | [Quiz Prep: MCP Server Building (20 Questions)](16-quiz-prep-mcp-server-building-20-questions) | — |
