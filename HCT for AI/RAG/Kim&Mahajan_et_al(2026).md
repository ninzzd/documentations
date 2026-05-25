# VectorLiteRAG: Latency-Aware and Fine-Grained Resource Partitioning for Efficient RAG

**Authors:** Junkyum Kim and Divya Mahajan

**Link:** https://arxiv.org/abs/2504.08930

## Summary
### Issues solved

- **CPU-based vector retrieval becomes the bottleneck in RAG** due to high ANN search latency on large vector databases.
- Moving retrieval fully onto GPUs causes **contention with LLM inference** for GPU memory, KV cache, and compute resources.
- IVF vector indexes exhibit **highly skewed access patterns**, where a few “hot” clusters dominate retrieval traffic.
- Query hit rates **vary significantly across batches**, causing tail-latency amplification during batched retrieval.
- Existing GPU caching systems **optimize throughput but ignore SLO** requirements.

### Solutions proposed

- VectorLiteRAG partitions IVF indexes across CPU and GPU **using latency-aware hybrid caching.**
- **Hot clusters are cached on GPUs** while cold clusters remain on CPUs to reduce contention and improve retrieval speed.
- A **performance + hit-rate model** predicts **optimal CPU/GPU partitioning** under given SLO constraints.
- Query-aware routing **sends only relevant cluster probes to each GPU shard**, reducing wasted GPU work.
- A **dynamic dispatcher forwards early-finished queries** immediately to reduce straggler delays (slowest request in a batch) and improve batching efficiency.
- **Runtime adaptive re-profiling** updates hot-cluster placement when workload distributions drift over time.
- Overall, the system **improves SLO-compliant throughput by up to 1.5×** without extra hardware.
