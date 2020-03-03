---
title: "L6: ILP"
date: 2020-03-02T20:05:47-05:00
---

# Instruction Level Parallelism (ILP)

ILP is the theoretical instructions per cycle (IPC) in the best case scenario disregarding almost all limitations.

What happens when we put all instructions in one cycle? We get close to 0 CPI which is good. But any decode/read cycle won't be accessing the correct value if it needed a previous instruction to finish before it. Forwarding only works to send ahead by one cycle (before that cycles operation), so it also doesn't solve the problem where we need results multiple stages earlier in the pipeline. Thus the dependent instruction **stall**s until the needed value can be forwarded.

## Dependencies

RAW (Data) dependencies are what effect ILP. When an instruction is relying on the result of a previous instruction it must wait one cycle before the result can be fowarded to it and it can execute. If the instruction isn't data dependent it can execute on the first instruction.

## False Depenencies

There are other dependencies, such as WAW (anti) which could cause the wrong result to end up in a register because an earlier instruction executes later. However, these **false dependencies** can be removed by register renaming. This separates the concept of **architectural registers** which are what the compiler writes/uses and **physical registers** which are *all* the possible location values can be stored. As the processor executes the code at the assembly level it does register renaming to obtain the correct values per the segment of code it's executing (based only on true dependencies). This is done via a **Register Allocation Table (RAT)**.

## Register Allocation Table (RAT)

Table that says which physical register has value for which architectural register. This is updated and used by the CPU as instructions are executed. As the CPU executes instructions the RAT is consulted to see where in physical memory the *current* value needed can be found. Then, when the instruction goes to write a value the RAT is updated with a new physical address that the register value can be found at.

Example:

When the instructions are compiled they refer to the architectural register.
```
ADD R1,R2,R3
SUB R4,R1,R5
XOR R6,R7,R8
MUL R5,R8,R9
ADD R4,R8,R9
```

When the instructions are executed they use the RAT to pull the location for their arguments and then choose an unused physical register to store the result. Then the RAT is updated with the location of that result.
```
ADD P10,P2,P3
SUB P11,P10,P5
XOR P12,P7,P8
MUL P13,P8,P9
ADD P14,P8,P9
```

||RAT|
|-|-|
|0|P0|
|1|P10|
|2|P2|
|3|P3|
|4|P14|
|5|P13|
|6|P12|

## ILP with Control Dependencies (aka branches)

In our perfect processor we also assume to have a perfect branch predictor. So we know which way the branch will go every time.

Branch instructions themselves may be truely dependent (they need the results of some register therefore can only execute later) but the instructions within a branch are not necessarily dependent, and can have no impact on ILP.

## ILP vs IPC

IPC is bound by real-world constraints and is very much hardware architecture dependent. ILP is theoretical and is meant to find the "perfect running conditions" for the program itself, regardless of the hardware. For good IPC we want a wide-issue (many executions at once) out-of-order processor allows for close-to-ILP level productivitiy. `ILP >= IPC`
