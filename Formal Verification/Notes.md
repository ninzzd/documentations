# Questions
- Why does memory usage fall after some iterations during backward rewriting? (ForMAt) (intermediate cancellations : decrease)
- Should there necessarily be a single SP for a verification problem? Why add SPs of individual output bits? (general SCA, ForMAt) (XOR: worst case expansion : can be verified easily : no formal proof as to why a design is hard to verify : open problem : why and which one explodes)
- Tools used for memory profiling? (ForMAt) (trivial, just output symbolic poly array length at each step?) (RevSCA : printed to command line : log file : system call within C++ : Amulete in C -  entire code, not optimized : fast-poly : future goal is to make open-source : RevSCA executable is available )
- Validity of assumption that MVM can be directly constructed from VVM modules parallely? (SCA-DP) (use any way of design : as long as SP is availale : should be verified : sequential circuits not verified, not explored yet : systolic arrays need to be unrolled to get combinatorial block to generate SP, but what if unrolling is very costly : come up with an example)
- Any explanation as to why MAC is less verifiable than DP-MVM? (SCA-DP, ForMAt) (term explosion is more?)
- You have mentioned future work on MVM and MMM?
- SCA and general formal verification for FP or systolic arrays? How to apply SCA and AIGs to sequential/pipelined designs?
- For large designs (FP) : use SCA to verify small modules (Mul, Add, Shift, etc) : contruct graph of modules (more complicated than AIG) : solve using backwards rewriting : Is recursive SCA a thing? What's so special about SCA? (partly answered in LBR-MAC, computationally intensive to find arbitrary optimal gate blocks) 
(quadratic or exponential growth whe trying to make a cut in the AIG to evaluate atomic blocks, goal is to find such sub blocks efficiently)
- Scope for novelty in SCA?Scope in engg/corporate research? Scope for modifications in SCA-based formal verification? (like why cancel terms only after complete backward rewriting, interleaved BR and term cancelations?) (what happens when monomials explode : memory is fully exhausted : intermediate term explosion is the biggest issue : which polynomial gets cancelled and which doesn't, scope for innovation : can we find info on why a circuit has a problem : prove and disprove verifiability : optimized designs are hidden, even bigger blow ups : floating-point : check if circuit is correct or not, can we find where the error might lie : what changes to : transform the circuit to an easier circuit : debugging - done by mixture of SAT and SCA)
- Design space exploration?

# Important Remarks
- Important : data structures, polynomial represenations, common sense
- Thesis : why are some architectures non-verifiable : is there some intuition : formalism : graph optimization techniques to verify non-verifiable designs
- Another idea : direct verification 
- Indicators directly from AIGs without needing to perform a verification pass for smaller bit-widths