# Late Breaking Results: Towards Efficient Formal  Verification of Dot Product Architectures

**Authors:**

**Link:**

## Premise
- MMM (Matrix-Matrix Multiply), MVM (Matrix-Vector Multiply) : Constructed from parallel VVM modules

## Experimentation
- Two DP configs: 
    - DT(mul)-SE(fsa)-RC(add-tree)
    - AR-RC-RC (same ids)
- Mul: 8-bit or 16-bit
- No of product terms: 2 to 128
- Final result: upto 4096 bits

## Observations
- Both verifiable
- Across both mul bit-widths, across all no. of product terms
- Identical SP size curves for small and large no. of product terms => SCA-AIG is scalable for DP archtiectures (of this type)

