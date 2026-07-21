---
title: Defining and Exposing Tools via MCP
parent: MCP Server Building
nav_order: 4
duration: "14:00"
---

# Defining and Exposing Tools via MCP
{: .no_toc }

*Video duration: 14:00*

The previous lecture showed a single `@mcp.tool()`-decorated function end to end. This one zooms into the part of that picture that deserves its own real treatment: how you actually design a tool's schema once you're authoring it specifically for exposure through MCP, and where that job is identical to — versus subtly different from — the tool-schema principles the Tool Layer already built.

## The core claim: nothing changes, and everything is amplified

Start from the point the MCP Architecture & Governance lecture already made and take it seriously: an MCP-exposed tool is, from the model's perspective, indistinguishable from a tool defined directly in application code. Every principle from the Tool Layer — precise naming, a description written with the whole tool set in view, structured fields as the default with prose reserved for genuine judgment calls, matching granularity to the shape of the underlying capability, honest and retryable-aware error responses — applies to an MCP tool with no modification whatsoever. If you remember one thing from this lecture, make it that there is no separate "MCP tool design" discipline distinct from ordinary tool design; there is only tool design, and MCP is one particular way a tool gets connected to a model.

What MCP amplifies, rather than changes, is the *stakes* of getting that design right, for two structural reasons. First, an MCP server is frequently shared infrastructure — one server, many clients, many organizations' worth of agents calling it — so a poorly designed tool's cost isn't paid once by whoever wrote it but repeatedly by everyone who connects. Second, a server author writing a tool for MCP exposure often has less visibility into what else will be in the model's tool set at call time than an engineer wiring a tool directly into one application does — a filesystem MCP server might be connected alongside a database server, a ticketing server, and a dozen others the server's author never anticipated, which makes the "write descriptions with the whole tool set in view" principle both more important and structurally harder to satisfy.

## Schema design specifics that matter more under MCP

A few schema-design questions come up with particular force once a tool is meant for MCP exposure rather than private, single-application use.

**Self-contained descriptions.** Because a tool's description is the primary signal a model uses to disambiguate it from every other tool a client happens to have connected — many of which the server author has no visibility into — an MCP tool's description needs to stand entirely on its own. A description that reads sensibly in isolation but only truly disambiguates itself against tools the author happened to imagine at the time of writing is a description that will misfire the moment it's deployed alongside some other server's similarly-named tool. Naming conventions that build in the underlying system's identity — `jira_create_issue` rather than a bare `create_issue` — are a common, practical mitigation, precisely because MCP tool sets get composed from multiple independent servers in ways a single-application tool designer never has to plan around.

**Schema-level validation as the server's actual safety boundary.** Because an MCP server is, functionally, a piece of code you may not have written yourself running with real credentials, the schema is doing double duty: it's the model's interface, and it's frequently the first line of defense against a malformed or adversarial call actually reaching the underlying system. Enums, ranges, and required fields aren't just usability niceties in this context — a tightly constrained schema is a genuine security control, since it's the mechanism that prevents an out-of-range or malformed argument from ever reaching the server's own implementation logic.

**Consistent error shape across every tool a server exposes.** A server that exposes a dozen tools with a dozen inconsistent error-response conventions makes the model's job harder in the aggregate even if any individual tool's errors are individually well-formed. Establishing one consistent shape — a `retryable: bool` field, a human-readable `message`, an optional `suggested_alternative` — across every tool on a given server pays off precisely because a model calling several tools from the same server benefits from a predictable error contract it only has to learn once.

## Versioning and evolving a tool once clients depend on it

A tool defined directly in one application's code can be changed freely, because the engineer changing it usually also controls every caller. An MCP-exposed tool loses that luxury the moment more than one client depends on it: renaming a parameter, tightening a previously-optional field to required, or changing what a given argument means can silently break every client and every model interaction relying on the old shape, with the server author frequently having no visibility into who's calling it or how. This pushes MCP tool design toward the same discipline API design has always needed once a contract has external consumers: additive changes (a new optional parameter with a sensible default) are safe; changes that alter the meaning or requirement-status of an existing field are not, and generally warrant a new tool name or an explicit versioning convention (`create_ticket_v2`) rather than a silent breaking change to `create_ticket` itself. A server author who treats a published MCP tool's schema as freely mutable internal code is setting up exactly the kind of silent, hard-to-diagnose breakage the naming and schema discipline in this module exists to prevent.

## Documentation as a first-class part of the tool, not an afterthought

Because an MCP tool's description is doing double duty — it's simultaneously the model's primary disambiguation signal and, often, the only documentation a human integrator reads before deciding whether to connect this server at all — it's worth treating that description with more care than an equivalent internal-only tool might get. A description that assumes context only the original author had (an internal system's jargon, an unstated precondition, an implicit unit of measurement) fails two audiences at once: the model, which has no way to ask a clarifying question, and the human evaluating whether this server is trustworthy and well-built enough to connect, which is itself a governance judgment call per the Tool Layer's server-vetting principle. A well-written MCP tool description reads as though it were the *only* documentation this tool would ever get, because for a model calling it mid-inference, it functionally is.

## Registering and returning results

Beyond the schema itself, an MCP tool's registration also determines what a successful result looks like coming back — typically text, but the protocol supports richer content types (images, structured data) where the underlying capability calls for it. The same discipline from the Tool Layer applies here too: a result should give the model exactly what it needs to proceed, not an unfiltered dump of whatever the underlying system happened to return. A `search_tickets` tool backed by a real ticketing system's API might receive a verbose, deeply nested JSON response from that system — returning that raw payload to the model, rather than a distilled, task-relevant summary, reintroduces the same "designed error message versus raw stack trace" problem the Tool Layer diagnosed for failures, applied now to successful results instead.

## A worked example: tightening a real tool

Take a `create_incident` tool exposed by an internal on-call MCP server, with a first-pass schema of `{ "title": string, "details": string }`. In isolation this looks reasonable, but deployed alongside a dozen other connected servers, a model might call it for anything from a genuine production outage to a minor cosmetic bug, because nothing in the schema distinguishes severity, and nothing in the description says what "incident" means in this specific organization's vocabulary. Tightening it — adding a required `severity` enum (`sev1` through `sev4`), a `service` field constrained to the organization's actual service registry rather than free text, and a description that states explicitly what threshold of impact warrants using this tool over a plain ticket — is the same schema-design discipline from the Tool Layer, just now applied under the added pressure of a tool that other teams' agents, not just the original author's, will be calling without any shared context about what "incident" was meant to mean.

**Why it matters for the exam:** Expect the exam to test whether you recognize that MCP-exposed tool design is not a separate discipline — a question describing a poorly-named or badly-schema'd MCP tool is testing the same Tool Layer principles, just relocated. Also expect scenario questions specifically about naming collisions across independently-authored servers and about schema-level validation as a genuine security boundary, since both are places where MCP's multi-server, shared-infrastructure reality genuinely raises the stakes beyond a single-application tool.
