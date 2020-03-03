---
title: "L2: Metrics and Evaluation"
date: 2020-03-02T20:01:47-05:00
---

# Metrics and Evaluation

## Performance

Latency is how long from start -> finish. Throughput is # of processes/second.

Comparing throughput is done via speedup = N:

`N = throughput(x)/throughput(y)`

whereas latency is:

`N = latency(y)/latency(x)`

In both instances we get "x is N times faster than y"

While performance is directly proportional to throughput, it is proportional to `1/latency`

---

## Measuring Performance

`Performance = 1/execution time`

However, execution time depends on user load
- this depends on user
- requires many programs that could be run
- user would have to willingly give performance data

**Benchmarks** are programs and user data agreed upon for performance measurement. Usually part of a benchmark suite to simlate different types of apps running on the hardware.

- Real applications
  - most representitave of realworld use
  - difficult to set up on new machine (because incomplete)
  - for actual machines
- Kernels
  - find most time consuming part of app
  - still can be too difficult on new machine (might not have compiler)
  - for prototypes
- Synthetic Benchmarks
  - behave like kernals but purposefully easier to compile
  - good for design studies
- Peak Performance
  - theoretical maximums (this is for marketing)

**Benchmark Standards** are set by independent organizations (made up of user groups, academics and manufacturers)
- TPC (DB, webservers, transaction processing)
- EEMBC (embedding processing)
- SPEC (raw processesors)
  - GCC (software dev workloads)
  - BWAVES, LBM (fluid dynamics)
  - PERL (string processing)
  - CACTUS ADM (physics/relativity)
  - XALANC BMK (XML)
  - CALCULIX, DEALL (diff eq)
  - BZIP (compression)
  - GO, SJENG (gaming)

**Summarizing performance** is done by averaging performance accross different applications. Not average speedup! Geometric means can be used on values and speed up.

---

## Iron Law of Performance

`CPU TIME = # instructions in program * cycles per instruction * clock cycle time`

We use these three metrics to calculate CPU time because it allows us to analyze different, important areas that each affect performance. Number of instructions in program is affected by the algorithm choice as well the compiler choice. Instruction set is also a factor. If the instructions are too simple it will take more of them to execute the same thing. Instruction set also affects the cycles, where simpler is a good thing that leads to fewer cycles. Processor design is the other factor for cycles. It also affects clock cycle time. Circuit design and transistor physics are the two other factors limiting clock cycle time.

Computer Architecture is affected by instruction sets and processor design.

**Unequal instruction times:**

Changes `# instructions in program * cycles per instruction` to be the sum of all the different types of instructions to get the total number of cycles.

`CPU TIME = SIGMA_i (IC_i * CPI_i) * TIME/CYCLE`

---

## Amdahl's Law

Used for speedup of part of the program or only some of the instructions and we want to know what the overall speedup is.

`SPEEDUP = 1 / ((1 - Frac_Enh) + (Frac_Enh/speedupAmnt))`

Where `Frac_Enh = % of original execution **TIME**`

Better to get small improvements on large portions of time rather than large improvements on small portions. e.g. **make common case fast**

---

## Lhadma's Law

Don't mess up uncommon case too badly while making improvements on common case.
