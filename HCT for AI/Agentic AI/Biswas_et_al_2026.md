# Sutradhara: An Intelligent Orchestrator-Engine Co-design for Tool-based Agentic Inference

**Authors**: [Anish Biswas](), [Kanishk Goel](), [Srivarshinee S](), [Jayshree Mohan](), [Alind Khare](), [Anjaly Parayil](), [Chetan Bansal](), [Ram Ramjee]()

**Link**: [Biswas et al. 2026](https://arxiv.org/abs/2601.12967)

## Summary
### Observations
- Agentic AI -> new perf bottleneck -> First Token Rendered latency
- tool calls contri 30-85% of FTR
- KV cache hit rates collapse despite substantial context reuse across iterations
- sequential orchestration wastes potential intrarequest parallelism

## Main Issues
- orchestrators and LLM engines operate as decoupled black boxes, preventing cross-layer optimizations

## Optimizations Suggested
-  overlap
tool execution with subsequent LLM prefill using tool-aware
prompt splitting (similar to RAGO, collocation)
- streaming tool execution to dispatch tools incrementally during decode rather than waiting for complete output
- orchestrator-aware cache management that uses semantic hints to improve hit rates and reduce
thrashing (preventing KV cache eviction)

## Results
- improves the throughput–latency trade-of
- sustains up to 77% higher load at the same median FTR latency, or reduces median FTR latency by up to 15% at the same load
- reducing end-to-end latency by up-to 11% on A100
GPUs

## Introduction
- architectural shift of LLM-only serving to agentic -> introduces a critical performance challenge that existing LLM serving infrastructure fails to address: latency explosions in user-perceived response times
- the cumulative latency
of multiple LLM calls interspersed with tool executions until
the first user-visible token

## Observations
- Tool execution dominates tail latency (P90), accounting for 30-85% of FTR latency while individual tool calls can exceed LLM prefill time
- Sequential orchestration
leaves parallelism unexploited, as 60-80% of each iteration’s prefill is tool-independent
- KV cache thrashing destroys reuse opportunities despite context reuse, as workload-agnostic LRU eviction thrashes shared prefixes when agentic requests execute concurrently