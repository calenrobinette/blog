---
title: "L3: Pipelining"
date: 2020-03-02T20:02:47-05:00
---

# Pipelining

## What is pipelining

Exactly like an oil pipe. Long latency for first action but all after 1st are continuous.

For processors this means:

I1, I2,... are different instructions

| Fetch | Read/Decode | ALU | MEM | WRITE |
|:-----:|:-----------:|:---:|:---:|:-----:|
| I1    |             |     |     |       |
| I2    | I1          |     |     |       |
| I3    | I2          | I1  |     |       |
| I4    | I3          | I2  | I1  |       |
| I5    | I4          | I3  | I2  | I1    |

Why wouldn't we get CPI (cycles per instruction) of 1 as expected?
- Initial fill (however CPI -> 1 as #instructions -> infinity)
- Pipeline Stalls (holds up the entire process)

**Control Dependencies**

Pipeline Stalls can be caused by instructions that jump which then cause all following functions in the pipeline to be **flushed** when/if the jump occurs.

`CPI_new = 1 + % chance of jump * (stage jump occurs at - 1)`

**Data Dependencies** 

`ADD R1,R2,R3`
`SUB R7,R1,R8`

Subtract has a data dependence on Add called a read after write (**RAW**) dependence because SUB reads R1 after ADD writes R1. Also called **Flow** (data flows forward) or **True** (R1 would be empty if ADD doesn't happen 1st) dependence.

`MUL R1,R5,R6`

Multiply must happen after Add so that R1 has the correct result. This is a write after write (**WAW**) or **Output** dependence. Also subtract must finish reading R1 before multiply stores into it. This is write after read (**WAR**) dependence. All of these are **False** or **Name** dependencies which have to do with overwriting and not about data flow.

---

## Dependencies and Hazards

Dependence != Pipeline stalls. There are many instances where there are dependencies that compute normally in order. If there is an incorrect execution as a result of a dependence it's known as a **Hazard**. If there are enough instructions between two instructions which are **True** dependent then the dependency isn't a hazard.

--

## Handling of Hazards

1. Flush Dependent Instructions
  a. This is done for Control Dependencies, since following instructions are unecessary/wrong and correct ones must follow
2. Stall Dependent Instructions
  a. For data dependencies, stops instruction until previous has completed what the dependency was waiting for
3. Fix values read by dependent instructions
  a. Once the value is needed for calculating (ALU stage) it can be obtained by the dependent instruction, "fixing" a missread. This doesn't always work and may still require a stall. Fowarding helps avoid a stall and saves a single cycle, thus **it is preffered to stalling**
  
---

## How many stages?

Ideal CPI is 1. More stages means more hazards which implies a higher CPI. However, it also means less work per stage which lowers the cycle time. For modern processors the balence between CPI and CycleTime is between 30-40 stages for performance purposes. For power efficiency more stages == more power draw.

Thus to balance CPI, cycle time and power efficiency the **ideal number of stages is 10-15** 
