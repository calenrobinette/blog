---
title: "L8: ReOrder Buffer"
date: 2020-03-02T20:07:47-05:00
---

# ReOrder Buffer (ROB)

## Execeptions in out-of-order execution

If something goes wrong during execution has to go to exeception handler. If other instructions are completed in that time, register values may be different then when the instruction originally started. This is also true in mispredicted branches altering register values before the branch was known to be mispredicted. Leading to the other side of the branch using the wrong values. Also, on the mispredicted side there could be an exception caused by an instruction that, we find out later, was never meant to be executed.

## Correct OOO execution

- Execute OOO
- Broadcast OOO
- Deposit values to registers **IN-ORDER**

The **ReOrder Buffer (ROB)** remembers the program order and keeps the results until they are safe to write.

## ReOrder Buffer

Table with N entries. Each entry has a Reg, Val, and Done. Reg is the Register Address it will be written to. Val is eventually filled with the value, and Done is weather or not it has has a value from an executed instruction. There is also a current issue pointer and commit pointer where ROB entries enter and are written from respectively.

With a ROB the Issue stage now gets a ROB entry after a RS for each instruction. The ROB entry is the next entry after the issue pointer and contains the R# that's the result for the instruction, as well as 0 for the Done bit. Then the RAT now points to the ROB entry.

Dispatch: Now the RS can be freed as soon as the inst is dispatched because the RAT points to the ROB not the RS#.

Write Result: when broadcast the result carries its ROB# instead of RS#, WR is always written to the ROB and the RAT is no longer updated because it's pointing to the value in the ROB#

**Commit:** Each cycle ROB entries are tested if Done and written to the register if true, also the RAT is updated to point at the register.

## Branch Misprediction Recovery

When a branch mispredicts, it is marked in the Val of the ROB and marked done. When the ROB goes to commit the branch result the handling happens

1. Issue pointer moved to Commit pointer after the branch entry
2. Done bits are reversed
3. ROB entries in the RAT are cleared and RAT points to Reg
4. Free RS and ALU

How this works well is because it restarts from program order to the point of the missprediction/exception. A big reason it works is that mispredictions/exceptions are treated like results until the ROB goes to commit them, then they are handled.

## Unified Reservation Stations

One big RS that feeds into multiple ALUs of different types. Dispatching logic becomes more complicated because the *type* of instruction needs to be considered to decide where it is sent.

## Superscalar

- Fetch >1 inst/cycle
- Decode >1 inst/cycle
- Issue >1 inst/cycle (must maintain program order, will stall if inst can't be issued)
- Dispatch >1 inst/cycle (dependent on number/type of ALUs)
- Broadcast >1 result/cycle (cost of RS++)
- Commit >1 inst/cycle

We have to worry about "weakest link" aka bottleneck.

## Out-of-order?

Fetch, Decode, Issue, and Commit are all in-order. Execute (Dispatch), and Write (Broadcast) are out-of-order.
