---
title: Topic 1: RAG and Knowledge Retrieval
parent: Extra Module: Beyond the CCA-F
nav_order: 2
duration: "0:26"
---

# Topic 1: RAG and Knowledge Retrieval
{: .no_toc }

*Video duration: 0:26*

The Context Layer module covered *why* you narrow a large corpus down before it reaches the model — the context window is finite, and more information isn't better information if most of it is irrelevant to the task at hand. RAG (retrieval-augmented generation) is the concrete "how" behind that principle. Documents are split into chunks, each chunk is embedded into a vector representation, and those vectors are stored in an index. At query time, the query itself is embedded, the index is searched for the nearest chunks, and only the top-k results are injected into context instead of the whole corpus.

That one-paragraph description hides most of the actual engineering work, which is where going deeper pays off. **Chunking strategy** matters more than people expect: chunks that are too large drag in irrelevant surrounding text and waste context budget, while chunks that are too small lose the surrounding context a passage needs to be understood correctly. **Hybrid search** — combining semantic (embedding) similarity with traditional keyword or BM25 search — consistently outperforms either approach alone, since embeddings are good at conceptual similarity but can miss exact terms (a product SKU, a legal defined term, a person's name) that keyword search catches trivially. **Re-ranking** adds a second pass that reorders an initial candidate set using a more expensive, more accurate model, trading a bit of latency for meaningfully better precision at the top of the results.

The discipline that ties all of this together is evaluating retrieval quality *separately* from generation quality. A great model fed the wrong chunks will still produce a confident, fluent, wrong answer — and if you only ever evaluate the final output, a retrieval failure looks identical to a reasoning failure. Serious RAG systems track retrieval precision and recall against a labeled test set independently of whatever the model does with what it retrieved, which is the only way to tell whether a bad answer came from bad context or a bad model response to good context.
