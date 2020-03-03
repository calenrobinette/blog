---
title: "L10: Compiler ILP"
date: 2020-03-02T20:09:47-05:00
---

# Compiler ILP

Compiler can help avoid limitied ILP because of dependence chains, as well as independent instructions that are far apart because the CPU can only see a "limited window" of instructions.

## Tree Height Reduction

Instead of creating cascading dependencies, can group instructions in more efficient ways:

```
R8 = R2 + R3 + R4 + R5

ADD R8,R2,R3
ADD R8,R8,R4
ADD R8,R8,R5
```
Instead could be:
```
ADD R8,R2,R3
ADD R7,R4,R5
ADD R8,R7,R8
```
This uses associativity, which not instructions have as a property

## Instruction Scheduling

Rearrange instructions at compliation. Moves non-dependent instructions to locations where other instructions are stalling due to execution. Have to "fix" values that use results innapropriately (example went from `SW R2,0(R1)` to `SW R2,-4(R1)` because an `ADDI R1,R1,4` was moved from after to before it)

Note: when an inst takes a number of inst, it stalls for n-1

## Scheduling and If-Conversion

If-Conversion allows code to be scheduled into different branches that wouldn't normally be allowed, because all the code is going to execute no matter what.

## Loop Unrolling

Unrolling once means to do two iterations in one loop cycle. Don't forget to change how the index moves (i = i - 2). 

One of the benefits is to reduce the number of total instructions executed. `ADDI` and `BNE` are executed half as many times after unrolling once. Reducing the #inst increases performance.

Another benefit is a direct CPI decrease: more instructions written out = more opportunity to schedule

## Downsides to Unrolling

1. Code Bloat: Code size is much much larger
2. What if #iterations unknown? (While loop)
3. What if the number of iterations is not a multiple of N (like 7)

## Function Call Inlining

If functions are simple, move the call into the compiled instructions to remove the function call and return. This helps by removing overheads (such as preparing function parameters) and also opens up scheduling. Larger improvements for smaller functions. Downside is the same as unrolling: code bloat.
