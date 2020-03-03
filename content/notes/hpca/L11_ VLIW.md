---
title: "L11: VLIW"
date: 2020-03-02T20:10:47-05:00
---

# Very Long Instruction Word

## Superscalar vs VLIW

||OOO Superscalar|In-Order Superscalar|VLIW|
|-|-|-|-|
|IPC|<=N|<=N|1 Large Inst some work as N "normal" insts|
|How do we find indep. inst.|Look at >N insts|Look at next N insts in program order|Just do next large inst|
|Hardware cost|Expensive|Less Expensive|Even Less Expensive|
|Help from compiler|Compiler *can* help|Needs help|Completely depends on compiler|

## VLIW: The Good and the Bad

Good:
- Compiler does the hard work
- Simpler Hardware
- Can be energy efficient
- Works well on loops and "regular" code

Bad:
- Latencies are not always the same
- Irregular Applications
- Code Bloat
  - In the case of dependent instructions in OoO processors, VLIW has to issue no ops which is wasted space per instruction
  - Can lead to 4x program size

## VLIW Instructions

- Has all the "normal" ISA opcodes
- Full predication support
- Lots of registers
- Branch hints from compiler to aid in BPred
- VLIW instruction "compaction"
  - Adding stops between instructions

## VLIW Examples

- Intel Itanium
  - Tons of ISA features
  - HW very complicated
  - Still not great on irregular code
- DPS Processors
  - Regular loops, lots of iterations
  - Excellent performance
  - Very Energy-Efficient

Note: good for adding -> Counting -> figuring out stuff
