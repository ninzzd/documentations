# Towards Understanding, Analyzing, and Optimizing Agentic AI Execution: A CPU-Centric Perspective

**Authors**: [Ritik Raj](https://scholar.google.com/citations?user=I8kUcx4AAAAJ&hl=en), [Souvik Kundu](https://scholar.google.com/citations?user=BHZqjebF-IIC&hl=en), [Ishita Vohra](https://scholar.google.com/citations?user=O0VsrxuUlJQC&hl=en), [Hong Wang](https://scholar.google.com/citations?user=0sx9_DYAAAAJ&hl=en), [Tushar Krishna](https://scholar.google.com/citations?user=P__ztgcAAAAJ&hl=en)

**Link**: [Raj et al. (2026)](https://drive.google.com/file/d/104j_zKznLMAfboFS9BpQcmNw6GQuxLGa/view?usp=drive_link)

## Characterization of Agentic AI

Orthogonal (uncorrelated) bases for classification:
1. Orchestration
2. Execution Path
3. Repetiveness

### Orchestration
**1. LLM-Orchestrated:** 
- LLM makes control flow decisions (eg. calling and scheduling tool execution, result aggregation) - probabilistic

- GPU-dominant

- (Refer to paper for example models)

**2. Host-Orchestrated (IMP):** 

- Python code, run on CPU, makes control flow decisions - traditional programming, deterministic

- LLM - stateless inference engine (LLM does not have different states, for text generation, orchestration reasoning, etc.)

- **CPU-dominant**

### Path
I don't see CPU-centric relevance 
**1. Static-Path:** 

- Predefined workflow, deterministic tool calls

- Specified in compile-time

**2. Dynamic-Path:**

- Adaptive construction of **execution graphs**

### Repetitiveness

**1. Single-Step:** 

 - Single inference pass, no environmental feedback

 - Eg. chain-of-thought inference, RAG,single-turn QA (not sure why they are called *agentic*) 

 **2. Multi-Step: (IMP)**

- Iterative refinements, extensive exploration

- Potential to be CPU-dominant - iterative tool execution - scattered inference - starved GPU

## Profiling and Analyses
### Setup
Sys1 - CPU, LP GPU
Sys2 - CPU, HP GPU (to isolate CPU-bottlenecks)
### Workloads
**Note: Inference - **SLMs** - better fit for agentic AI than monolithic LLMs - reduced parameters, similar or better performance


1. **Toolformer** - tool-agnostic (crazy) - LLM model decides which tool to call and when (dynamic-path) (LLM-orchestrated)- benchmark : WolframAlpha (may be repetitive)
    - No official source-code/API from Meta, several open-source implementations/paper-replications available on Github (eg. )

2. **SWE-Agent** - (LLM-orchestrated) inference - generates code and Bash commands - sends to CPU to execute in terminal - (multi-step) (dynamic-path)
    - Open-source, cloned from Git, tried running, SWE-agent seems to be working fine, I don't have credits for OpenAI API 
3. **[ChemCrow](https://github.com/ur-whitelab/chemcrow-public)** [[Paper](https://arxiv.org/abs/2304.05376)] - solve chemistry problems - molecular property prediction - uses ReAct (reasoning and acting) (LLM-orchestration) (dynamic-path) (multi-step)
    - Open-source, GitHub repo available, must clone and test
4. **RAG** - using Haystack (idk much abt this) - uses ENN (**CPU-heavy**) - 115 GB doc corpus
    - open-source, available on [git](https://github.com/deepset-ai/haystack)
5. **Web-Augmented Agent** - Langchain (same) - web-search -> lexical summary (CPU-based) -> LLM-inference (static-path) (CPU-orchestrated ig) (single-step)

### E2E Latency Analysis

**Conclusions:** 
- Tool-processing - significant chunk of E2E - HP GPU shifts bottleneck towards CPU even more (especially when LLM-inference and tool-exec latencies - similar) - thus, similar E2E with HP/LP GPU


### GPU Throughput Analysis
**Methodology**
```math
vLLM\_throughput = BS 
* \frac{T_{in} + T_{out}}{t_{sec}}
```
$T_{out}$ - no. of output tokens
$T_{in}$ - no. of input tokens
$BS$ - batch size

**Conclusions**
- throughput **increases proportionally** with batch size (increased utilization)
- rate of throughput increase **reduces** (**saturation**) for large batches
- PagedAttention - reduces fragmentation - does not improve GPU memory bandwidth limitations - that's the GPU bottleneck

### CPU Parallelization
**Conclusions:**
- Premature saturation of CPU throughput for small increase in CPU parallelization, inefficient CPU parallelization => degraded GPU utilization
## Proposed Solutions

### COMB
CPU-Aware Overlapped Micro-Batching
- Large batches -> split into smaller batches
- CPU micro-batches - smaller than GPU micro-batches
- Optimal batch size - at the edge of CPU throughput saturation

### MAS
Mixed Agent Scheduling
- Supports **minority** workloads
Traditionally:
If requests < lim: execute
Else: Queue
lim is globally defined

Here, requests are of distinct types with separate limits.
Scheduling is bifurcated:

If # cpu_req < cpu_lim: execute on cpu
Else: queue

If # gpu_req < gpu_lim: execute on gpu
Else: queue

implemented with two parallel exec queues



