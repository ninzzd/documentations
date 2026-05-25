# HedraRAG: Coordinating LLM Generation and Database  Retrieval in Heterogeneous RAG Serving

**Authors:**

**Link:** https://arxiv.org/abs/2507.09138v1

**Citations:** 10

## Revisit Later
- 4.5 : Dynamic Graph Transformation and  Scheduling 
- 5 : Implementation

## Summary
### Issues
- RAG systems : no longer one-shot retrieve-generate systems : multi-hop, iterative retrieval, query rewriting/decomposition, document reranking/compression, agentic workflows (*almost exactly matches classifications in [1][1]*)
- LLM inference: iterative, vector search : one-shot : disparity created pipeline inefficiency, temporal phase imbalance
- Variable length stages : dynamic, irregular structure of complex RAG systems : stage interleaving, request variability : resource contention
### Solutions
- **RAGraph** : DAG with generate nodes and retrieval nodes: runtime reordering, splitting, overlapping of graph nodes
- **Fine-grained partitioning** (retrieval of 200 vectors broken into clusters of 20, similar with LLM) : overlapped generation-retrieval : increased parallelism : fine grained sub-stage partition for both generation and retrieval
- **Semantic Similarity Optimization**: in iterated retrieval, searched vectors in succesive iterations may be semantically similar (nearby) : **cache previous search results**, **reorder cluster search**,
**prioritize likely-useful clusters** : reduced search latency
- **Speculative execution**: start generation before complete retrieval : increased parallelism
- **GPU vector caching**: during retrieval : some vectors clusters more accessed than others : **hot vectors** : keep hot vectors in GPU cache, ready to be passed to inference, remaining in CPU

## Obervations and Motivation
- **fine grained sub-stage partition** for both generation and retrieval 
    - retrieval : single-cluster search
    - decode : single or small group of generated tokens
 - **dynamic load-aware alignment**
    - adjusts sub-stage boundaries based on real-time workload
- **intra-request semantic similarity** (locality) (solves issue of varying fine-grained batch sizes and determination of sizes)
    - previously retrieved passages/vectors are closer to current retrieval request than top-5 or top-20 vectors (within top-5 or top-20) (iterative retrieval, multiple retrievals in pipeline)
    - 22-50% of token sequence : yields good enough similarity/semantic meaning for next retrieval stage
    - hence: **workload-aware speculative execution**, **locality-based cluster reordering** (semantic similarity and locality allows for fine-grained sub-stage partitions)

- **GPU-Acc vector search : index access skewness**
    - large databases : retirieval is bottleneck : offload some clusters to GPU : GPU vector search usually standalone : GPU memory occupied with LLM weights and KV cache
    - exploit locality
    - user requests centered around similar topics or scopes tend to generate query embeddings spatially close to a shared subset of clusters in the index.
    - cache only hot clusters into GPU mem for GPU vector search
    - partial GPU index cache with async updates
    - advantage of GPU for vector search : much higher throughput

## HedraRAG

### Key Ideas

### RAG Components
- CPU-GPU hybrid retrieval engine

### Design
- uses **graph-abstraction of RAG**
- series of graph transformation techniques to optimize serving
- **encapsulates optimization techniques**: stage parallelism, intra-request similarity, inter-request skewness; **as graph transformations** : node splitting, reordering, edge addition, dependency rewiring (graph part is just an abstraction, not real optimization)
- dynamic optimizations/graph transformations and scheduling

**Fine-Grained Sub-Stage Pipelining**
- aligning the short-latency, multi-step decoding with the long-latency, singlestep retrievals,
- coordinated dynamic batching : fundamental units of scheduling and execution
- **dynamic time-budgeting** method based on **retrieval request** : to determine cluster size and no. of decoded tokens for sub-partition size : each **retrieval request are incrementally added until a maximum time budget mb is reached** : retrieval centric : much higher workload variance than generation
- modelled as **node-splitting** in RAGraph

**Intra-Request Semantic Search**
- locality-based bservations : 
    - The search results of v′ tend to be included within the search results of v with a larger top-k.
    - When the search results of v are in a cluster set Hv, the results of v′ also tend to be located in Hv
    - The search results of v′ tend to be located in clusters of C ∩ C′
- **caching** : set of larger top-k results of v (20 in practice) are stored in a local cache for future reuse
    - search for v′ is first attempted in the local cache of v : major overhead is computation of semantic similarity, memory access latency is negligible
    - earlier termination in ANNS by up to 28%

- **Speculative Execution**
    - early-terminating property of ANNS to enable speculative execution

    - **speculative generation** : generation from partial retrievals (from few clusters, not all) : overlapped generation with continued retrieval: **if complete retrieval produces same output passages/vectors as speculative retrieval**, preoceed: if not match, re-generate with results from complete retrieval (**rollback** condition)

    - **speculative retrieval** : retrieve vectors/passage from **small initial set of generated tokens** (occurs parallely (pipelined) with generation) : real retrieval occurs at end of generation : **results of speculative retrieval guides real retrieval** (**result caching** from speculative retrieval)

    - **adaptive speculative strategy** based on both **workload dynamics** and **semantic similarity** (knowing when to start speculative retrieval/generation is tough) : triggered when **CPU/GPU system throughput of the next sub-stage is underutilized** with $\frac{T_{curr}}{T_{max}} < \tau$ ($T_{curr}$ : current throughput **emperically estimated from no. of request + prefill tokens**, $T_{max}$ : peak system throughputs) : run speculative executions **until throughput ratio reaches threshold $\tau$ at each sub-stage**

    - graph abstraction : modify the dependency structure between sub-nodes : add edges for speculative exec, overlapping with original sequential path with rollback

**Partial GPU Indexing**
-  GPU search (with FAISS-IVF) : high throughput, very fast : vector search is L2 distance computation, highly parallelizable : final distance results, aggregated to find top-K vectors (retrieval output)
- Cluster skewness => hot clusters accessed most often : cache in GPU along with LLM weights and KV cache : SIMT-accelerated retrieval in GPU, multi-threaded retrieval in CPU (parallelized across hot clusters) => hybrid CPU-GPU retrieval engine
- GPU retrieval, CPU retrieval and PCIe transfers : all parallelized : maximized hardware utilization
- Offline benchmarking : determine optimal KV cache size : arg-max that maximizes the lesser of generation-throughput and retrieval-throughput (on GPU) (varied across all request arrival rates (requests-per-second for retireval and generation))
- graph abstraction : further parallelization within retrieval sub-stage nodes/sub-nodes

**Dynamic Graph Tranformations and Scheduling**


## Key Takeaways

## Drawbacks
- Uses FAISS-IVF conveniently : parallelizable : hence GPU-acceleration, hot-cluster-caching, etc. : **what about ScaNN, HNSW, and other ANNS algorithms**?
- **Does not report end-to-end model accuracy**. Speedup and throughput gain are great, but cannot come at huge costs of accuracy : HedraRAG does not change semantics and workloads, only schedules and parallelizes them

<!-- 
Below contains reference links
-->
[1]: /HCT%20for%20AI/RAG/Jiang_et_al_2025.md
