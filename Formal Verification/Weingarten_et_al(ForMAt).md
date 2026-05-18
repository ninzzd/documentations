# ForMAt: Formal Verification of Scalable Multiply and Accumulate Units

**Authors**: 

**Link**:

**Contributions:**
- MAC-Gen : automated generation of MAC architectures
- MAC-Verifier : formally verifies MAC isntances using SCA (symbolic computer algebra)
- Analysis and classification of MAC architectures w.r.t verificability
- Define scalability indicators
- Including verifiability as a design parameter

## SCA-Based Verification
- Specification Polynomial (SP): 
- Gate Polynomial (GP): 

## MAC-Gen
- Verilog-code generator for a certain configuration of MAC, as described below
- 196 variations
- MAC-Gen inputs 3 stage params, generates Verilog, Yosys converts Verilog to AIG (And-Inverter Graph)
- AIG : input to MAC-Verifier
- Area and delay calculations : Cadence Genus Solution utilizing the ASAP 7 7.5-track standard cell library for 7-nm technology node
(graph of areaXdelay shows dadda-tree architectures to be optimal)


### Multiplier (General Architecture)
1. Partial Product Generator (PPG) 
2. Partial Product Accumulator (PPA) 
3. Final Stage Adder (FSA)

### Choices for Each Subcomponent
**PPG**
- Simple Unsigned
**PPA**
- Array
- Counter based Wallace tree
- Dadda tree
- Wallace tree
**FSA/MSA** (MSA : MAC Stage Adder : Adds AxB with S (accumulate stage))
- Ripple carry
- CLA
- Ladner-Fischer
- Kogge-Stone
- Brent-Kung
- Carry Skip
- Serial prefix

## MAC-Verifier
- Figure 6 is extremely important
- **Verifiability:** If backward rewriting of SP can be achieved **within given timeout period**
- **Goal:** Given *G* (AIG), output TRUE if SP is zero polynomial, else FALSE

### Scalability Indicators
- $\phi_1 = \frac{MaxPoly}{|SP_{init}|}$ : ratio of peak to initial mem usage (peak achieved at $i=ts$)
- $\phi_2$ : ratio of area above $|SP_{init}|$ to area below $|SP_{init}|$


### Experimental Insights
**\*\*Note:** **L2 distance from origin** of **area-delay** graph : optimilatiy metric for a design
- **Almost all 8-bit** MACs : **verifiable**
- **40%** 16-bit MACs : **verifiable**
- Muls AR and DT : best for verifiability
- All FSAs **except carry-skip adder** (CS gives timeout for AR and DT) are suitable, some cases of **KS** are also bad (with RC, SE and CL)
- **No locality** in terms of **area** and **delay** for verifiable designs
- 5 out of 10 most optimal designs : verifiable
- **(\*\*IMP)** VT : Verification Time : **proporational** to $\phi_1$, $\phi_2$, *MaxPoly*
- Non-verifiable designs : comparitively very large $\phi_1$, $\phi_2$, *MaxPoly*
- Hence : $\phi_1$, $\phi_2$ : if low (around (1,0)) => high likelihood of verifiability for higher bit-widths
(observation supports this)

### Questions and doubts
- Memory-spike : large no. of terms after substitution : Why does memory fall after substitution steps?
- Is source-code available?
