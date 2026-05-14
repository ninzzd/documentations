# RAGO: Systematic Performance Optimization for Retrieval-Augmented Generation Serving

**Authors**: [Wenqi Jiang](https://scholar.google.com/citations?hl=en&user=0gT0jzkAAAAJ&view_op=list_works), [Suvinay Subramanian](https://scholar.google.com/citations?hl=en&user=54KhKdEAAAAJ&view_op=list_works), [Cat Graves](https://scholar.google.com/citations?user=QJY5iFoAAAAJ&hl=en), [Gustavo Alonso](https://scholar.google.com/citations?user=fkNjVcIAAAAJ&hl=en), [Amir Yazdanbakhsh](https://scholar.google.com/citations?user=Vdu_sqwAAAAJ&hl=en), [Vidushi Dadu](https://scholar.google.com/citations?user=vMb6d1MAAAAJ&hl=en)

**Link**: [Jiang et al. 2025](https://arxiv.org/abs/2503.14649v2)

## RAG Paradigms (Imp)

### Hyperscale Retrieval
 - Instead of large LLM-only serving -> RAG with large doc corpus (GB to TB) + small LLM model

 - **Bottleneck:** **Retrieval** (due to very large corpus)


### Long-Context Sequence Processing

## RAGSchema
Defines:
- execution flow of RAG pipeline
- configuration of components (vectorDB and LLM are mandatory, reranker and rewriter and database encoder are optional) (<ins>configs</ins> include \#params,queries_per_vec,\#database_vec, etc)

## Experiment Methadolody
### Models
- **4** LLMs: Llama-3 1B, 8B, 70B, and 405B
- Models are **quantized** to **8-bit integer** (\#params=\#bytes in mem)
### Database
- Database contains **64 billion passages**, each encoded as a **768-dimensional vector**
- Product quantization (PQ) to compress each vector to **96 bytes**(1 byte encodes 8 dimensions)
- ScaNN (Scalable Nearest Neighbours): fanout is **4K vectors** per node, **3-level tree index**

### LLM Sequence Lengths
- Derived from QA datasets
- For simplicity, take expected/average number, eg. typical question length (in tokens), nearest-neighbour count, avg length of input prompt, etc.

### System Setup
(numbers aren't important, just to give an idea of scale)
**General**
- Data center model
- 16-32 servers
- 64-128 XPUs (4 XPUs per server)
- \#servers > 16 : sufficient host memory for database

**XPU**
- ML Acelerator : TPU v5p : 96 HBM : 2.7 TB/s mem BW : 459 TFLOPs
- XPU-CPU interconnect : 600 GB/s across 6 links (100 GB/s per link)

**CPU**
- AMD EPYC Milan : 96 cores : 384 GB RAM : 460 GB/s mem BW

### Simulation Setup

**Inference**
- **Model sharding** for multi-XPU inference (each XPU : subset of inference operators)
Sharding:
    - pipeline parallelism
    - tensor parallelism
- Inference : abstracted as **sequence of operators**
- [**Timeloop**](https://accelergy.mit.edu/timeloop.pdf)-like simulation/modelling
- Execution time of each operator : calculated using **roofline model** (see paper for more details, just some simple formulae)

**Retrieval**
- Search process : modelled as : sequence of vector scans at each level of a multi-level tree
- **One thread per query** : multi-queries parallelized : **MT** 
- Cost of each retieval operator : roofline model : funcs of batch size, \#cores, per-core CPU throughput and mem BW (again, refer to PDF for formula, not required)
- Servers with distributed retrieval : each server : shard of vectorDB : independent datasets
- To feed sim params : benchmark per-core CPU throughput and mem BW : on smaller dataset with same vec-fanout (4K vec per node) (check PDF for exact values obtained)
- Obtained values : calibrated to 64 bil. dataset (5.6 TiB vectorDB) using internal projection dataset (no comprehende)

**Inference-Retrieval Comms**
- Negligible in practice
- \#tokens-transferred x bytes-per-token
- Throughput: Comms >> inference
