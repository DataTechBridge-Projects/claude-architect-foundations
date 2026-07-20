---
title: Putting the "Provenance" in Data Provenance
parent: The Context Layer
nav_order: 13
duration: "0:56"
---

# Putting the "Provenance" in Data Provenance
{: .no_toc }

*Video duration: 0:56*

The word "provenance" originally describes the documented history of an object — a painting, an antique, a legal title — the chain of custody that lets you trust it's what it claims to be. Data provenance borrows that idea directly, and the borrowing is more literal than it sounds: a fact in a context window deserves the same chain-of-custody treatment as a painting up for auction. Where did it come from, who or what handled it, when was it last authenticated, and does its documentation actually travel with it or live somewhere separate and easy to lose? An art dealer would never say "trust me, it's real" without papers. A context window that states a fact with no attached source, timestamp, or trust level is making exactly that unsupported claim — just in prose confident enough that nobody thinks to ask for the papers.

Push the analogy one step further and it explains why provenance has to be *attached*, not just *available somewhere*. A painting's provenance documentation isn't useful sitting in a separate filing cabinet across town while the painting itself hangs on an auction wall with no reference to it — the whole point is that the papers travel with the object, so that anyone evaluating it at the moment of decision can check them without a separate hunt. The same is true of a fact in an agent's context: a source and timestamp logged in some external audit trail, disconnected from the fact itself as it appears in the window, might satisfy a compliance requirement after the fact, but it does nothing to help the model — or a human reviewer — weigh that fact correctly *at the moment it's being used*. Provenance that isn't co-located with the claim it supports isn't really provenance in any functional sense; it's just a record that something was, at some point, checkable.

A concrete illustration: imagine two versions of the same context entry. One reads "the customer's shipping address is 44 Birch Lane." The other reads "the customer's shipping address is 44 Birch Lane (source: account database, verified 12 minutes ago)." Both convey the same core fact, but only the second is actually usable the way provenance is meant to be used — if a later step in the same session encounters a conflicting address from an older email, the first version gives the model nothing to weigh the conflict with, while the second immediately tells it which source should win and why.

This lecture is deliberately the short, name-the-idea companion to the "Data Provenance and Conflicts" lecture later in this module, which works through the full mechanics — what metadata to attach, how to resolve conflicting sources, and how provenance survives (or doesn't) summarization and compaction. Treat this one as the thirty-second version worth remembering even if you forget everything else: no papers, no trust, no exceptions.
