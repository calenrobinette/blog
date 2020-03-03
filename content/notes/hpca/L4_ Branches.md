---
title: "L4: Branches"
date: 2020-03-02T20:03:47-05:00
---

# Branches

## Branch in a pipeline

example:

`BEQ R1,R2,Label` means that if R1 and R2 are equal PC "jumps" to where ever label is. So:

R1 != R2 -> PC++
R1 == R2 -> PC++ + offset to get to Label

When the latter happens all other instructions in the pipeline behind the branch are now wasted cycles and the new useful instructions need to start being fetched.

## Branch Prediction Requirements

BPred must correctly guess:
- Is this a branch?         } --> Is this a taken branch?
- Is it taken?              }
- If it's taken what is the target PC?

## Branch Prediction Accuracy

CPI = cycles per instruction

CPI = 1 + ^Mispreds^/~inst~ * ^penalty^/~mispred~

The above can be summarized as the base 1 CPI plus how often we mispredict times the number of cycles wasted when we mispredict (Pipeline architecture)

For a BPred with 50% accuracy for branches, 100% accuracy for all other instructions, and 20% of all inst are branches:

With 3rd stage branch resolution
`CPI = 1 + 0.5 * 0.2 * 2 = 1.2`

With 10th stage branch resolution
`CPI = 1 + 0.5 * 0.2 * 9 = 1.9`

When the BPred accuracy is increased to 90%

3rd stage
`CPI = 1 + 0.1 * 0.2 * 2 = 1.04`

10th stage
`CPI = 1 + 0.1 * 0.2 * 9 = 1.18`

Branch prediction is very important!

Speedup for 3rd stage is `1.2/1.04 = 1.15` whereas 10th stage is `1.9/1.18 = 1.61`

### Note from quiz

`ADD` takes 2 cycles to load and then decode, not 1

---

It is always better to predict than to not predict and have every instruction take a minimum of 2 cycles.

## Predict Not-Taken

For the 80% of non-branch instructions it is 100% accurate
For the remaining 20% that are branch instructions if we estimate that 60% are taken that means a total of 12% of instructions are taken branches and incur wasted cycles. Thus there is 88% accuracy.

`CPI = 1 + 0.12 * penalty`

## How do we make better predictons

We need to find ways to identify if an instruction is a branch, whether or not that branch is taken, and what the offset is to add to the PC to get to the jump the branch wants to take.

Branches tend to act the same historically, so we can use past branch behaviors to inform our branch predictor.

## BTB - Branch Target Buffer

Index PC for branches into a table (BTB) with the PC the branch jumps to associated to it. If the branch at that PC goes through the pipeline and jumps to a different PC, the table is updated.

The BTB needs to be small to keep fetching down to a single cycle. If we indexed every PC in a program on a 64 bit system, it would be far too large.

In order to keep the BTB small it is reserved for instructions likely to be computed (as in a loop). Additionally, only the 10 least significant bits of the current PC are used as the indexing entry. This works because the PCs will be relatively close due to only storing instructions "close-by"

## Direction Prediction

Similar to the BTB the Branch History Table (BHT) but just contains a single bit. 0 if not a taken branch (PC++) or 1 if it's a taken branch (use BTB). If the BHT is wrong, it is updated. This allows the BTB to only be reserved for taken branches thus reducing its size.

1-bit predictors are good for always taken or always not taken. They're pretty good for 90% T/NT. Anything other than that they're bad.

### 2-bit predictor (2BP, 2BC (2-bit counter))

More signifigant bit is the prediction bit (0 not taken, 1 taken), least signifigant bit is the hysteresis or conviction bit. 00 is Strong Not-Taken, 01 is Weak Not-Taken, etc. If the branch ends up being not taken the number is decremented if possible and incremented if the branch was taken.

The initialization state can completely mess up the prediction accuracy of certain patters. For example if the predictor is init as 00 and the pattern is T N T N it will be 50% accurate. There are other similar situations, but it generally isn't worth worrying about except for specific situations.

3BP and 4BP have much higher costs without seeing much more accuracy. Only situationally useful, doesn't match usual cases in actual programs.

## History-based predictors

This allows for pattern recognition by setting a 2BP for each of the historical situations. For a 1-bit history with 2BC we have:

| 1 bit for the previous branch | BC for 0 history | BC for 1 history |
|-------------------------------|------------------|------------------|
| Taken or Not Taken            | SN, WN, WT, ST   | SN, WN, WT, ST   |

The above is useful for T, N, T, N but not for something like N, N, T, N, N, T. In order to recognize longer patterns, more history bits are necessary (specifically a 2 bit history predictor for the mentioned pattern). N-bit history predictors can be used for all patterns of length <= N + 1. For N-bit history predictors the costs increase rapidly.

`Mem req = N + 2^N * 2` and this is per entry (each branch). The `2^N` is the number of 2-bit counters needed for N-bit history.

With a pattern of length N+1 there can only be N+1 histories. Example: T T N N has T T N N, T N N T, N N T T, and N T T N. However, this requires eight 2-bit counters!

## History with Shared Counters

In order to reduce space we can share the counters. N-bit history requires 2^N counters per entry, but only ~N counters are actually used. The solution is to use multiple entries (branches) per 2BC, this **can** lead to memory conflicts, but rarely does if there are enough counters.

How it works:
- Take some number of signifigant bits from the PC for each branch instruction and use it as an index for **Pattern History Table** (PHT)
- Store N bits of history (no bit counters, just N N T T)
- Take the history from the table and combine it with the PC (with an XOR usually) and use that to index for **Branch History Table** (BHT) who's entries are single 2-bit counters.
- The BHT 2BC tells us if the branch should be taken or not taken.

The idea is to store a single 2BC per history + branch combination, which is a lot less than `2^N`. However, it is possible for multiple PC x PHT combinations to map to the same 2BC in the BHT. The memory requirements shrink to: `2^PC-bitsused * N-history + 2^N-history * 2` This is much less than before because the `2^PC-bitsused * N-history` replaces the total number of branches.

In this system, simple patterns (TTTT, NNNN, NTNT) require very few 2BCs (1, 1, and 2 respectively). Leaving more room for longer non-repeating patterns that require many 2BCs.

This is known as **PShare**. Where each branch *should* have it's own **P**rivate history in the Pattern History Table which combined with the branches program counter maps to the Branch History Table of **Share**d counters. PShare does well when the branches previous behavior is predictive of how it will act in the future (NTNT loops, 8-iteration loops).

Another type of history predictor we can use is **GShare** which uses a single **G**lobal history with it's own BHT of **Share**d counters. Every branch decision effects the single global history entry. This is good for correlated branches. This happens surprisingly often. Think of using `if (var == condition)` and `if (!var == condition)`. The history for the first if statement can be used to correctly predict the results of the second if statement accurately.

Modern processors use both GShare and PShare.

## Tournament Predictor

Using a **Meta-Predictor** table of 2BCs we store which predictor (GShare or PShare) should be used for each branch (similar to accessing the PHT and BHT, we use the PC to index into the meta-predictor). Each predictor is trained on every branch result. The meta-predictor is only trained (incremented up or down) if one predictor is correct and the other isn't in which case it becomes biased towards the correct predictor.

## Heirarchical Predictors

Tournament: 2 good predictors

Hierarchical: 1 good, 1 ok predictor

This is done to reduct maintenance costs by mostly using the ok predictor with low memory usage, and only using the good predictor on more complicated branches. This means that the okpred is updated every decision and the goodpred is only updated if the okpred didn't work.

Example:

Pentium M uses a 2BC predictor, Local History, and Global History

Steps for a branch
1. Checks global tag array to see if branch is predicted by global history
2. If yes, uses GShare, if no checks local tag array to see if branch is predicted by local history
3. If yes, uses PShare, if no uses 2BC
4. Whatever predictor the branch **is already stored in** gets updated, if 2BC was used but was wrong and the branch isn't in the local predictor, we add the branch to the local predictor to make sure it's used next time. Same goes from local -> global (if local mispredicts and branch isn't in global -> branch added to global so it's used next time)

## Return Address Stack (RAS)

Most branches have static target adresses they jump to and can be accurately predicted using a BTB. However, function returns (where the PC goes after a function is completed) are harder to predict where they're supposed to goto (They are always taken however). This is because functions are *usually* created to be called multiple times in a program.

RAS is a predictor for function returns. When a function is called the return address (function address + 4) is added to a stack. When the function return's that address is popped from the stack and used. If the RAS is full, the PC is stored by wrapping around. This allows for the x most recently called functions to have an accurate return address, which is usefull for nested functions that call the inner functions multiple times.

```
billy() {
    bob() {
        steve() {
            // code
        }
        
        // more code
        
        steve() {
            // even more
        }
    }
}
```

There are 2 methods to tell if an inst is a function return: predictor or **predecoding**. Predictor (just a like a branch predictor) is very accurate and not very memory intensive. Just need a single bit predictor. Predecoding TOO TIRED FINISH TOMORROW
