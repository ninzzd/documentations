# RAGO: Systematic Performance Optimization for Retrieval-Augmented Generation Serving

**Authors**: [Wenqi Jiang](https://scholar.google.com/citations?hl=en&user=0gT0jzkAAAAJ&view_op=list_works), [Suvinay Subramanian](https://scholar.google.com/citations?hl=en&user=54KhKdEAAAAJ&view_op=list_works), [Cat Graves](https://scholar.google.com/citations?user=QJY5iFoAAAAJ&hl=en), [Gustavo Alonso](https://scholar.google.com/citations?user=fkNjVcIAAAAJ&hl=en), [Amir Yazdanbakhsh](https://scholar.google.com/citations?user=Vdu_sqwAAAAJ&hl=en), [Vidushi Dadu](https://scholar.google.com/citations?user=vMb6d1MAAAAJ&hl=en)

**Link**: [Jiang et al. 2025](https://drive.google.com/file/d/1Nx5VC8VrV-GNbN8vveYBWExUYeH1grwm/view?usp=drive_link)

## RAG Paradigms (Imp)

### Hyperscale Retrieval
 - Instead of large LLM-only serving -> RAG with large doc corpus (GB to TB) + small LLM model

 - Match or surpass perf of large LLMs without retrieval

 - **Bottleneck:** **Retrieval** (due to very large corpus)


### Long-Context Sequence Processing