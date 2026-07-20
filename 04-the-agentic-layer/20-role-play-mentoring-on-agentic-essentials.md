---
title: Role Play: Mentoring on Agentic Essentials
parent: The Agentic Layer
nav_order: 20
---

# Role Play: Mentoring on Agentic Essentials

**Mentee:** I've been asked to design an agent that handles customer refund requests end to end — read the request, check the order history, decide whether it qualifies, issue the refund, and email the customer. My instinct is to just build one agent with all the tools it needs and let it run the whole thing in one loop. Is that wrong?

**Mentor:** Not wrong on its face, but let's pressure-test it against what we covered. Start with the orchestration loop itself — walk me through what happens, step by step, if this agent decides wrong.

**Mentee:** It would... call the refund tool with bad arguments, I guess, and the refund goes out anyway.

**Mentor:** Right. And what category of action is "issue a refund"? Reversible, compensable, or irreversible?

**Mentee:** Compensable, I think — you could claw it back or issue a correction.

**Mentor:** Compensable, sure, but expensive and embarrassing to compensate after the fact. That's exactly the signal for where a safeguard belongs. Not everywhere — you don't want a human approving every step, or you've just built an expensive form. But right before the irreversible-feeling action.

**Mentee:** So an approval gate right before the refund actually gets issued.

**Mentor:** Exactly. Now, second question — is this one agent's job, or does it decompose?

**Mentee:** I was going to keep it as one big task.

**Mentor:** Look at the steps: read the request, check order history, decide eligibility, issue the refund, send the email. Do they share the same tools and the same stakes?

**Mentee:** No — reading the request and checking history are read-only and low-risk. Issuing the refund is the risky one. Sending the email is almost as low-risk as the reads.

**Mentor:** That's your decomposition signal right there — a change in tools, and a change in blast radius, both pointing at the same boundary. So what does that suggest architecturally?

**Mentee:** Maybe: one step that gathers everything and proposes a decision, a gate before the refund executes, and then a final step that sends the confirmation.

**Mentor:** Good. Now — does any of that need to be a separate subagent, or is it all one agent moving through those steps?

**Mentee:** I was tempted to make "check order history" a subagent, since it might involve a lot of searching across systems.

**Mentor:** What's the actual justification for that, from the module?

**Mentee:** Context isolation — if it takes a lot of exploration to piece together the order history, I don't want all that search noise sitting in the main agent's context. I just want the conclusion.

**Mentor:** Right, and what would you hand that subagent, and what would you withhold?

**Mentee:** Hand it the order ID and what it's looking for — prior refunds, delivery status, that kind of thing. Withhold... the customer's actual refund request text? It doesn't need that to look up history, and giving it doesn't help.

**Mentor:** And its tool access?

**Mentee:** Read-only lookup tools. It definitely shouldn't have the refund tool itself.

**Mentor:** That's the whole design, and notice you got there without inventing anything new — you applied the decomposition signal, matched the safeguard to the action's reversibility, and scoped the subagent to exactly what it needs and nothing more. One more thing, though — this is presumably not a one-shot interaction. What happens across many refund requests over a day?

**Mentee:** That's session management, right? If it's all one long session, the history balloons — hundreds of unrelated refund cases stacking up in context.

**Mentor:** So?

**Mentee:** Each refund request is probably its own task, maybe its own session even, rather than one giant session that never resets. There's no reason case forty needs case one still sitting in context.

**Mentor:** That's the architecture. Notice you didn't need a new concept for any of it — just the loop, applied carefully at each layer.
