# HedraRAG: Coordinating LLM Generation and Database  Retrieval in Heterogeneous RAG Serving

**Authors:**

**Link:**

## Summary
### Issues
- RAG systems : no longer one-shot retrieve-generate systems : multi-hop, iterative retrieval, query rewriting/decomposition, document reranking/compression, agentic workflows (*almost exactly matches classifications in [1][1]*)
- LLM inference: iterative, vector search : one-shot : disparity created pipeline inefficiency, temporal phase imbalance
### Solutions
- **RAGraph** : DAG with generation nodes and retrieval nodes: runtime reordering, splitting, overlapping of graph nodes
- **Fine-grained partitioning** (retrieval of 200 vectors broken into clusters of 20, similar with LLM) : overlapped generation-retrieval : increased parallelism
- **Semantic Similarity Optimization**: in iterated retrieval, searched vectors in succesive iterations may be semantically similar (nearby) : **cache previous search results**, **reorder cluster search**,
**prioritize likely-useful clusters** : reduced search latency
- **Speculative execution**: start generation before complete retrieval : increased parallelism
- **GPU vector caching**: during retrieval : some vectors clusters more accessed than others : **hot vectors** : keep hot vectors in GPU cache, ready to be passed to inference, remaining in CPU

<!-- 
Below contains reference links
-->
[1]: /HCT%20for%20AI/RAG/Jiang_et_al_2025.md
