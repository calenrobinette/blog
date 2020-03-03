---
title: "L5: Predication"
date: 2020-03-02T20:04:47-05:00
---

# Predication

Using the compiler to avoid hard to predict branches. Branch prediction guesses where the branches are going with no penalty if they are correct. A missed prediction throws away many cycles of work.

**Predication** is where both sets of instructions are loaded (taken/not taken) and only one is thrown out ensuring waste is only 50% of total work. The advantage of predication is very niche, since if a BPred is fairly accurate it accrues very little penalty on average whereas predication *always has a penalty*.

- Loops should always be done via BPred since they are easy to predict
- Funcation call/returns don't have multiple directions of work to do so predication makes no sense.
- Large if-then-else will guarantee to waste however long one side of the branch making it just as bad, if not worse, than BPred *if there's a missprediction*.
- However, Predication is useful for small if-then-else because the branching paths of instructions could be smaller than throwing out the entire pipeline in the case of a missprediction.

So predication should be used for id-then-else statements with less instructions than what's thrown out in a missprediction multiplied by the chance of misspredicting. This is because predication's cost is guaranteed and must be compared to the average cost of missprediction.

## If-Conversion

How the compiler creates the code to be executed on both paths.

```
if (cond) {
    x = arr[i];
    y = y + 1;
} else {
    x = arr[j];
    y = y - 1;
}
```

This translates into:

```
x1 = arr[i];
x2 = arr[j];
y1 = y + 1;
y2 = y - 1;
x = cond ? x1 : x2;
y = cond ? y1 : y2;
```

But to ensure that we're not just substituting the same problem by translating x = cond... and y = cond... into two branches, we have to come up with a `MOV` instruction that only operates if a condition bit is set. This is known as **Conditional Move**.

For MIPS: `MOVZ R_d, R_s, R_t` -> `if(R_t == 0) { R_d = R_s }` and `MOVN` is the for the opposing condition `R_t != 0`. So, by using both we can get the `x = cond ? x1: x2;` we're looking for.
x86: `CMOVZ`, `CMOVNZ`, `CMOVGT`, etc. This allows for the removal of the `R3 = cond;` statement below because the condition is checked by the asm instruction.

```
R3 = cond;
R1 = x1;
R2 = x2;
MOVN x, R1, R3
MOVZ x, R2, R3
```

**quiz note:** `BEQZ R1, Else` - if R1 is zero jump to `Else:`, otherwise proceed to the next instruction

## MOVZ/MOVN Performance

For BPred: `Cost = (#inst_path_A + #inst_path_B) / 2 + %MisPred * MisPredCost`
For If-Conversion: `Cost = #inst`

**quiz note:** `accuracy = 100 - MisPred`

## MOVc summary (Conditional MOVs)

It's important to note, this approach needs *compiler support*. The advantage is that hard-to-predict branches are removed and replaced with instructions that are guaranteed to execute. This does add cost in the form of additional registers needed and more instrctions executed for both paths.

However, to increase performance we can make all instructions conditional so that the register is not always written to and we no longer need MOV to select the results. Instead the results are only calculated/written if the condition is met. This is known as **Full Predication**

## Full Predication Hardware Support

Same instructions as the If-Conversion, just add a condition bit (qualifying predicate) to every instruction. This leads to the removal of the MOV instructions

```
Mp.EQZ    P1,P2,R1
(p2) ADD1 R2,R2,1
(p1) ADD1 R3,R3,1
```

`Mp.EQZ` checks if R1 is zero and sets p1 to true and p2 to false if R1 is zero. The opposite if R1 is 1.

**quiz note:** calculate total cycle cost for branch inst vs if-conversion then compare the difference. This is the "allowable" penalty. The allowable/actual penalty for misspred gives mispred% -> accuracy = 100 - mispred%
