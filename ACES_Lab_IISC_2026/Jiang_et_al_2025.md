# RAGO: Systematic Performance Optimization for Retrieval-Augmented Generation Serving

**Authors**: [Wenqi Jiang](https://scholar.google.com/citations?hl=en&user=0gT0jzkAAAAJ&view_op=list_works), [Suvinay Subramanian](https://scholar.google.com/citations?hl=en&user=54KhKdEAAAAJ&view_op=list_works), [Cat Graves](https://scholar.google.com/citations?user=QJY5iFoAAAAJ&hl=en), [Gustavo Alonso](https://scholar.google.com/citations?user=fkNjVcIAAAAJ&hl=en), [Amir Yazdanbakhsh](https://scholar.google.com/citations?user=Vdu_sqwAAAAJ&hl=en), [Vidushi Dadu](https://scholar.google.com/citations?user=vMb6d1MAAAAJ&hl=en)

**Link**: [Jiang et al. 2025](https://arxiv.org/abs/2503.14649v2)

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

**XPU** (XPU-C is default, see paper)
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

## RAG Paradigms (RAGSchema)
**\*\*Note**: performance measured with QPS/chip vs TTFT latency pareto lines
### Hyperscale Retrieval
 - Instead of large LLM-only serving -> RAG with large doc corpus (GB to TB) + small LLM model
- config: RETRO
- One retrieval operation stage before prefill and decode, multiple independent tokens of the same request are sent for querying and retrieval (batched querying)

**Exp Observations:** 
- Bottlneck:**Retrieval** (due to very large corpus)
-  Increasingly dominant for:
(1) smaller LLM, 
(2) multi-query retrievals 
(3) better inference accelerators (shifts bottleneck to CPU) 
(4) shorter prefix and decode sequence lengths
(5) higher retrieval quality.
 
    **Advantages**
    - **Max throughput (QPS/chip) of RAG 8B = 1.5 x Max throughput of LLM-only 70B**
    
    **Drawbacks**
    - **retrieval overhead** and the **need for longer prompts** to integrate retrieved information (512 tokens in RAG versus 32-token questions in LLM-only systems)
     => <ins>QPS/Chip gain is not directly proportional to the reduction in parameter size</ins>

**Dependence of Retrieval on LLM Size**
- For small LLMs (8B) : **query per retrieval : inversely scales with QPS** 
(as query counts double, QPS nearly halves due to increased retrieval demands) : retrieval is the bottleneck **for any number of queries per request**
- For large LLMs (80B) : **initial limiter : inference** : **beyond 8 queries per request : retirieval dominates**

**Dependence of Retrieval on \#tokens**
- Contribution of retrieval overhead to total latency : **drops with increase in prefix and decode lengths** (inference dominates for longer token sequences)
- Adjusting the prefix and decode lengths : **unequal changes** in the percentage of retrieval time (not exactly linear) : reason : prefix inference : **inherently faster than decoding the same number of tokens** : autoregressive nature of decoding.

**Dependence of Retrieval on RAG config**
- highly sensitive to the percentage of database vectors scanned per search.
- fundamental trade-off between **retrieval performance** and **quality** : scanning more vectors improves quality but reduces performance (obvious ngl)
- tradeoff influenced by data distribution (ANN performance depends on the geometry of the embedding space, which depends on data distribution)
- increasing the scanned database vectors significantly amplifies the proportion of time spent on retrieval

**Summary of Observations**
- Increased throughput compared to LLM-only: rate decreases with increase in retrieval overhead
- retrieval overhead: increases with decrease in LLM size
- retrieval overhead: reduces with longer prefill and decode lengths
- retrieval overhead: significantly increases with increase in $P_{scan}$ (% of database searched per query) 

### Long-Context Sequence Processing
- LLM Q&A based on lengthy doc (around 100K tokens) : instead of including entire doc in prompt : treat doc as a knowledge DB :  retrieve only the relevant information for the question
- Substantial reduction in prompt size : similar response quality
- Setup requires **DB encoder** : converts doc to vector DB : DB is **orders of mag. smaller than in hyperscale** (see paper for eg.)

**Exp Setup**
- Context lengths : from 100K to 10M tokens : encoded DB : 1K to 100K vectors, with each chunk/paragraph sized at 128 tokens and small overlaps between chunks
- sentence transformer model : 120 M parameters : encode the passages : generating 768-dimensional embeddings (vector DB)
- **Brute force kNN** (k-th nearest neightbours search) **instead of ANN** : ANN has high indexing costs associated with newly generated embeddings

**Exp Observations**
- Retrieval : **minimal role**
- New bottleneck : **DB vector encoding** : encoder must process **significantly longer context** compared to long context LLM
**Dependence on Context Length**
- As the context length increases : RAG performance gradually degrades : cuz of **increasing cost of context encoding** : **despite LLM prompt-truncation from retrieval** (offsets the gain)
- encoding time scales with context length, despite the relatively small encoder applied : due to the sheer volume of data processed
- retrieval time is minimal even when using brute-force search due to small database
- *potential soln*: caching the generated embedding for potential reuse
### Iterative Retrieval

### Query Rewriter and Reranker
