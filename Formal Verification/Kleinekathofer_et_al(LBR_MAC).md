# Late Breaking Results: Efficient Formal Verification  of Highly Optimized MAC Units

**Authors:**

**Link:**

## Issues with Optimized MAC Designs
- Intermediate term explosion : exponential growth of SP during substitution : designs become unverifiable
- Optimized designs : complicated gate connections : leading to term explosion

## Experimental Setup
- Behavioral code for highly optimized MAC : sent to Yosys and ABC to generate optimized AIG
- AMD Ryzen 7 : 40 GB MM

## Proposed Changes
### Dynamic Selection
- **Dynamic selection/substitutation algorithm** guided by **polynomial size**
- Aim: **controlled** polynomial size growth
- Availability of gate : when all successors of its output have already been substituted.
- For each available gate:
    - Perform substitution : measure polynomial growth $ = \frac{PolySize_{i}}{PolySize_{i-1}}$
    - If PolyGrowth > thres (1.3) : revert substitution
    - Else, keep the substitution
    - If no available gate satisfies, start substitution on gate with least PolyGrowth

### Polarity Optimization
- **Negate the polarity** of each signal  in each monomial, do for all signals
- Canonical form is maintained : every molynomial must have signals according to their single global polarity
- Find the optimal polarity for each signal with **smallest PolySize**
- Eg. 
```math
    OR(a,b,c) = a + b + c - ab - bc - ac + abc \; (size=7)\\
    = 1 - \overline{a}\overline{b}\overline{c} \; (size=2)
```
- Substitute smallest poly

## Results
- RevSCA and Transform-RevSCA : **lower verification time for small bit-widths** (2-bit to 4-bit)
- 5-bit onwards (**upto 15-bit**) : **proposed outperforms SOTA** : SOTA results in memory timeout (MTO) beyond only 10-bit
- Scalable verification solution

## Some Important Points
- Does not group gates into blocks : computationally expensive : problematic for ckts with hidden internal structures
