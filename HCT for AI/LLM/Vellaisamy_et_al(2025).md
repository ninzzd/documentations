# Characterizing and Optimizing LLM Inference Workloads on CPU-GPU Coupled Architectures

**Authors:** Prabhu Vellaisamy,Thomas Labonte, Sourav Chakraborty, Matt Turner∗, Samantika Sury∗, John Paul Shen

**Link:** https://arxiv.org/abs/2504.11750

## Summary

### Issues addressed

* Existing LLM inference studies **do not deeply characterize fine-grained CPU↔GPU operator/kernel interactions** across loosely-coupled (PCIe A100/H100) and closely-coupled (GH200) systems. 
* It is **unclear whether closely-coupled CPU-GPU architectures universally outperform traditional PCIe systems** for latency-sensitive inference workloads. 
* Existing **profiling tools lack detailed operator-to-kernel dependency analysis** needed to identify CPU launch overheads, GPU queuing delays, and kernel-level bottlenecks. 
* **Low-batch inference workloads remain CPU-bound** due to **kernel launch overheads** and **underutilized GPUs**, especially in tightly integrated systems like GH200. 
* Existing **kernel fusion methods** are mostly **domain-specific** (FlashAttention) or graph-level (torch.compile), **lacking a general runtime-driven fusion** recommendation framework. 

### Solutions / contributions

* Introduces **SKIP**, a PyTorch-based profiler that builds **operator-kernel dependency graphs** and **analyzes fine-grained CPU-GPU offload behavior**. 
* Proposes **TKLQT (Total Kernel Launch and Queuing Time)** as a metric to classify workloads into CPU-bound vs GPU-bound regions more accurately than prior “framework tax” approaches. 
* Shows that GH200 achieves much better large-batch inference performance (up to 1.9x–2.7x faster prefill latency for Llama-3.2-1B) because of high-bandwidth memory and tighter CPU-GPU coupling. 
* Demonstrates that GH200 remains CPU-bound for up to 4× larger batch sizes than PCIe systems, revealing CPU launch overhead and Grace CPU single-thread performance as bottlenecks for low-batch inference. 
* Introduces a **proximity-score-based kernel fusion recommendation framework** that automatically identifies deterministic kernel chains suitable for fusion from runtime traces. 
* Shows that scalable kernel fusion can theoretically provide large latency gains in CPU-bound regions by reducing kernel launches (up to 2.7× for GPT2 and 6.8× for XLM-Roberta-Base). 
* Concludes that future CC/TC architectures need stronger CPUs or smarter scheduling/fusion strategies to fully exploit high-bandwidth GPU subsystems.  
