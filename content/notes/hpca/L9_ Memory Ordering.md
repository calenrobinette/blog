---
title: "L9: Memory Ordering"
date: 2020-03-02T20:08:47-05:00
---

# Memory Ordering

How to do `LOAD` and `STORE` in an out-of-order processor.

# When does memory write happen?

For `STORE` it happens at **commit**. Has to be done at that point for the same reasons commit happens in program order.

Where does load get data? From a load-store queue. (This seems like ROB for memory insts)

## Load-Store Queue (LSQ)

Structure like ROB for `LOAD` and `STORE` with 4 parameters

|L/S|ADDR|VAL|C|
|-|-|-|-|
|Load or Store|Address|Value (if any)|Complete?|

If a `LOAD` address matches a previous `STORE` address in the LSQ the value is copied from the `STORE` LSQ entry (**Store-to-load forwarding**)

If an inst *doesn't have an address* there are options on how to handle further instructions:
- Do everything in-order
- Wait for all previous store addresses to be known (not completed)
  - This is because we can compare with them and know weather or not we need their val or need to go to memory
- Go anyway
  - In this case `STORE`s will also check if their address matches any further in the list and those instructions then go into recovery because they have the wrong values.

Go anyway is usually used

## Store to Load Forwarding

LOAD: Which earlier store do we get value from?
STORE: Which later load do I give my value to?

When loads commit they **only then** store in the register. When store commits it **only then** writes to the data cache/memory.

## LSQ, ROB and RS

All instructions get a ROB entry. LSQ acts as a RS for LOAD/STORE. So if there isn't a ROB entry **and** a LSQ entry available the IQ stalls.

Execute LOAD/STORE:
1. Compute address
2. Produce value (for STORE 1 and 2 can be done in either order)
3. Write Result (LOAD)
4. Commit (Free ROB and LSQ entries)
  a. STORE: write to memory
