# Towards Understanding, Analyzing, and Optimizing Agentic AI Execution: A CPU-Centric Perspective

**Authors**: Ritik Raj, Souvik Kundu, Ishita Vohra, Hong Wang, Tushar Krishna

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

### Workloads
**Note: Inference - **SLMs** - better fit for agentic AI than monolithic LLMs - reduced parameters, similar or better performance


1. **Toolformer** - tool-agnostic (crazy) - LLM model decides which tool to call and when (dynamic-path) (LLM-orchestrated)- benchmark : WolframAlpha (may be repetitive)

2. **SWE-Agent** - (LLM-orchestrated) inference - generates code and Bash commands - sends to CPU to execute in terminal - (multi-step) (dynamic-path)

3. **ChemCrow** - solve chemistry problems - molecular property prediction - uses ReAct (reasoning and acting) (LLM-orchestration) (dynamic-path) (multi-step)

4. **RAG** - using Haystack (idk much abt this) - uses ENN (**CPU-heavy**) - 115 GB doc corpus

5. **Web-Augmented Agent** - Langchain (same) - web-search -> lexical summary (CPU-based) -> LLM-inference (static-path) (CPU-orchestrated ig) (single-step)

### E2E Latency Analysis

**Conclusions:** 
- Tool-processing - significant chunk of E2E - HP GPU shifts bottleneck towards CPU even more - thus, similar E2E with HP/LP GPU
