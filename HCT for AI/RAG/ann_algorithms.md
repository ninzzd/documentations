# ANN (Approximate Nearest Neighbor) Algorithms
### References
- ANN Benchmarks:  https://ann-benchmarks.com/index.html

## HNSW (Hierarchical Navigable Small World)
- Ref: https://www.pinecone.io/learn/series/faiss/hnsw/
- **Graph-based** proximity index. Vertices linked by closeness (e.g. Euclidean).
- Combines **skip list hierarchy** (Pugh 1990) + **NSW greedy routing** (Malkov 2011–14).

### Core Idea
- Multi-layer graph: top layers = sparse, long-range links (coarse routing); layer 0 = dense, short-range (accurate).
- Search: enter top → greedy hop to nearest neighbor → local minimum → drop layer → repeat → answer at layer 0.
- O(log N) search. Upper layers route; layer 0 dominates compute.
- **Upper layers**: Simple greedy routing (beam width = 1) tracking a single nearest node per layer.
- **Lowest layer (Layer 0)**: Beam search of width `efSearch` maintaining global priority queue of closest nodes to yield top-$K$.

### Systems Profile
- **Memory-latency bound** — pointer-chasing via graph edges. Cache misses dominate, not compute - irregular memory access - major issue with GPU implementations.
- **High memory footprint** — full graph + raw vectors resident. ~1–2 GB for 1M 128-d vectors, M=32. Linear in N×M.
- **Key params**: `M` (connectivity/memory), `efSearch` (beam width → recall/latency knob).

### Acceleration
- **Software prefetching** neighbor lists: ~10-30% speedup (HW prefetch ineffective — data-dependent access).
- **SIMD distance kernels**: M distance computations per hop → AVX2/AVX-512/NEON vectorizable.
- **PQ compression**: reduce footprint, improve cache hit rate. Trades recall. (`IndexHNSWPQ`).
- **IVF+HNSW composite**: HNSW as coarse quantizer for IVF scan.

### Parallelizability
- **Inter-query**: trivial — no shared mutable state. Primary parallelism axis.
- **Intra-query**: hard — greedy traversal is sequential (each hop depends on previous).
- **GPU**: poor for single-query (sequential + irregular access). Good for **batched multi-query** throughput.
- **Sharding**: partition graph across nodes, merge results. Cross-shard edges need replication.

## DiskANN (Vamana)
- Ref: Subramanya et al., "DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node" (NeurIPS 2019)
- **Graph-based**, optimized for **SSD-resident** datasets that exceed RAM.

### Core Idea
- **Vamana graph**: single-layer graph (not hierarchical like HNSW). Built with a greedy construction that maximizes **graph navigability** while bounding max out-degree (parameter `R`).
- Search: beam search from a medoid entry point. Wider beam than HNSW (compensates for single layer).
- Key insight: fewer layers → **sequential disk reads** are feasible. HNSW's multi-layer random hops are disk-hostile.

### Systems Profile
- **Designed for SSD I/O** — graph layout optimized to minimize random reads. Vectors stored on disk; compressed (PQ) representations cached in RAM for candidate pruning.
- **Memory**: only PQ codes + graph metadata in RAM. Raw vectors on SSD. ~4-8 GB RAM for 1B vectors vs. ~hundreds of GB for in-memory HNSW.
- **Latency**: single-digit ms per query on SSD (vs. sub-ms for in-memory HNSW). ~5-10× slower per query, but enables **1000× larger datasets** per node.
- **Key params**: `R` (max out-degree), `L` (beam width at construction), `Ls` (beam width at search).

### Acceleration
- **SSD layout optimization**: graph + vector data co-located on disk sectors. Reduces random I/O.
- **PQ-based candidate pruning**: compute approximate distances from RAM-cached PQ codes → only fetch raw vectors from SSD for top candidates. Amortizes disk reads.
- **SIMD distance kernels**: same as HNSW for the compute portion.
- **NVMe parallelism**: overlap multiple async disk reads

### Parallelizability
- **Inter-query**: trivial, same as HNSW.
- **Intra-query**: beam search is more parallelizable than HNSW greedy — multiple candidates expanded concurrently, disk reads pipelined.
- **GPU**: not a natural fit (SSD-bound, not compute-bound).
- **Sharding**: natural partition by data subset. Each shard is an independent DiskANN index.

## IVF (Inverted File Index)
### Core Idea
- **K-means clustering**: find centroids of clusters, determine cluster boundaries (construction of IVF)
- **Coarse search**: Identify nearest `nprobe` clusters in which or around which query vector `x` is located
- **Exact search**: Run exact scoring on the vectors in these `nprobe` clusters, find nearest neighbors


## ScaNN (Scalable Nearest Neighbour Search)

### Core Idea
- **Partitioning**: similar to IVF, determine a subset of all clusters to be searched (clustering done using k-means)
- **Approximate Scoring**: Key contribution of ScaNN, unlike PQ or SQ, **anisotropic quantization** - minimizes dot product error, not euclidean error, hence minimum recall loss due to quantization
- **Pruning**: Dissimilar vectors after approximate scoring are removed
- **Exact reranking**: Compute scored based on exact euclidean distances on unquantized vectors for smaller subset of vectors, post pruning