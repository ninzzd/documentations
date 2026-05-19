# Questions
- Why does memory usage fall after some iterations during backward rewriting? (ForMAt)
- Tools used for memory profiling? (ForMAt) (trivial, just output symbolic poly array length at each step?)
- Validity of assumption that MVM can be directly constructed from VVM modules parallely? (SCA-DP)
- Any explanation as to why MAC is less verifiable than DP-MVM? (SCA-DP, ForMAt) (term explosion is more?)
- You have mentioned future work on MVM and MMM?
- SCA and general formal verification for FP or systolic arrays? How to apply SCA and AIGs to sequential/pipelined designs?
- For large designs (FP) : use SCA to verify small modules (Mul, Add, Shift, etc) : contruct graph of modules (more complicated than AIG) : solve using backwards rewriting : Is recursive SCA a thing? What's so special about SCA? (partly answered in LBR-MAC, computationally intensive to find arbitrary optimal gate blocks)
- Scope for novelty in SCA?Scope in engg/corporate research? Scope for modifications in SCA-based formal verification? (like why cancel terms only after complete backward rewriting, interleaved BR and term cancelations?)
- Design space exploration?