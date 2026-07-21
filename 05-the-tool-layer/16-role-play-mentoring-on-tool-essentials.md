---
title: "Role Play: Mentoring on Tool Essentials"
parent: The Tool Layer
nav_order: 16
---

# Role Play: Mentoring on Tool Essentials

**Mentee:** I'm building tools for an agent that manages our internal knowledge base — create articles, update articles, search articles, archive articles. I gave it four tools that map directly to those four verbs. Feels clean to me, but the model keeps calling `update_article` when it should be calling `create_article`, and vice versa.

**Mentor:** Before we touch the schemas, show me the descriptions.

**Mentee:** `create_article`: "Creates a new knowledge base article." `update_article`: "Updates an existing knowledge base article." That's it.

**Mentor:** Read those back as if you were the model, with no other context, deciding between them for a request like "add a troubleshooting section to the VPN setup guide."

**Mentee:** Oh — that's ambiguous. "Add a section" could sound like creating something new, even though it's really modifying an existing article.

**Mentor:** Right. So what's actually missing?

**Mentee:** The descriptions don't say when to use one versus the other. I need something like: "create_article: use this only when no article on this topic exists yet — check with search_article first. update_article: use this to modify an existing article; requires the article's ID."

**Mentor:** Better. Notice what you just did — you wrote the description with the *other* tool explicitly in view, not in isolation. That's the core habit. Now, second question: how does `update_article` fail when the model passes a bad ID?

**Mentee:** Right now it just throws whatever the database driver returns — something like `Error: no rows updated`.

**Mentor:** Walk through what the model does with that.

**Mentee:** Honestly, probably nothing useful. It doesn't know if the ID was wrong, if the article was deleted, or if it should just try again.

**Mentor:** And is retrying with the same ID going to work?

**Mentee:** No — if the ID's wrong, retrying identical arguments changes nothing.

**Mentor:** So that's a non-retryable failure, and right now your error doesn't say so, and it doesn't give the model anywhere to go. What would you rather return?

**Mentee:** Something like: "No article found with ID 'kb-4471'. This is not retryable with the same ID — use search_article to find the correct article first."

**Mentor:** That's the whole error-design lecture in one sentence — specific, marked non-retryable, and it names the actual recovery path. Now, one more thing. What happens if `search_article` also comes up empty?

**Mentee:** Then... I guess the agent's stuck? It doesn't have a way to just ask a human.

**Mentor:** That's your gap. Not every failure should resolve through more tool calls. If search comes back empty and there's still no clear ID, that's exactly the point to stop and escalate rather than let the agent guess at IDs or spiral through more searches with slightly different terms.

**Mentee:** So I need some kind of budget — after search fails once or twice, stop and report back instead of continuing indefinitely.

**Mentor:** Exactly, and that's a harness-level decision, not something you should leave to the model to figure out fresh every time it hits this wall. Last thing — you've got four tools for four verbs. Is that the right granularity, or could some of these merge?

**Mentee:** I was worried about that too. But create and update take genuinely different arguments — create needs a title and body, update needs an ID plus whatever's changing. Merging them into one tool with an optional ID field feels like it'd just recreate the ambiguity we started with.

**Mentor:** Which is exactly right — that's not over-fragmentation, that's four operations that are genuinely distinct in both purpose and shape. The problem was never the tool count. It was that the descriptions didn't do the disambiguation work, and the errors didn't give the model anywhere to go. Fix those two things first, always, before you touch granularity.

<!-- prevnext:start -->

---

| [&larr; Previous: Quiz Prep: The Tool Layer (20 Questions)](15-quiz-prep-the-tool-layer-20-questions) | [Next: The Workflow Layer &rarr;](../06-the-workflow-layer/) |
|:---|---:|

<!-- prevnext:end -->
