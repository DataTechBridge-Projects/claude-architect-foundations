---
title: These Are Local!
parent: The Workflow Layer
nav_order: 7
duration: "0:31"
---

# These Are Local!
{: .no_toc }

*Video duration: 0:31*

Not every workflow decision needs to scale beyond the team that made it. Whether code review requires two agentic passes or one, whether a `CLAUDE.md` calls out a quirky internal tool, whether a Skill exists for this team's particular report format — these are legitimately local choices, shaped by one repo's risk tolerance, one team's tooling, one project's history of near-misses. Trying to force every workflow convention into a single org-wide standard usually produces a bloated, generic policy that fits no one well. Some things should be centralized; plenty of things should just be local, and that's fine.

Consider two teams at the same company. A payments team, still recovering from a near-miss where a generated migration silently corrupted a staging table, might mandate two independent agentic review passes plus a mandatory human sign-off before any database-touching change merges — an appropriately paranoid process for that team's specific scar tissue. A marketing site team shipping copy changes and static content has no comparable history and would be actively worse off adopting that same two-pass ceremony; it would just slow down low-stakes work for no corresponding safety benefit. Neither team is "wrong" — they're both applying this module's stakes-and-reversibility reasoning to their own actual risk profile, and that profile genuinely differs by team.

This is also exactly why Claude Code supports nested `CLAUDE.md` files at multiple directory levels rather than forcing one flat, org-wide file, as covered in the Configuring CLAUDE.md lecture: a root file can hold what's genuinely true everywhere, while a `services/payments/CLAUDE.md` or a `services/marketing-site/CLAUDE.md` holds the local color that only makes sense in that corner of the codebase. Skills work the same way — nothing requires every team to draw from one shared Skill library, and a Skill tuned to one team's exact report template is often more useful than a generic one every team compromises around.

None of this means centralization is never right. Security constraints, legal and compliance requirements, and anything with cross-team blast radius genuinely do need a consistent, org-wide standard — you don't want one team quietly deciding customer PII handling is "locally" their call. The judgment this lecture is building is knowing which category a given convention falls into: does inconsistency here create real risk or confusion across teams, or does it just reflect that two teams reasonably do things differently? Most workflow conventions are the latter, and treating them as if they needed a company-wide committee to bless them is its own kind of waste.

<!-- prevnext:start -->

---

| [&larr; Previous: Workflow Design](06-workflow-design) | [Next: CI/CD Integration &rarr;](08-ci-and-cd-integration) |
|:---|---:|

<!-- prevnext:end -->
