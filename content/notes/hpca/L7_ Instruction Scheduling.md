---
title: "L7: Instruction Scheduling"
date: 2020-03-02T20:06:47-05:00
---

# Instruction Scheduling

How to execute real world programs faster, even with data dependencies. Tomasulo's algorithm was and still is one of the most efficient ways to take the theory of ILP and physically recreate it in hardware.

## Improving IPC

Using out-of-order processing to obtain >1 IPC

- How to handle control dependencies => highly accurate branch prediction
- False data dependencies => register renaming (RAT)
- True data dependencies => out-of-order execution
- Structural dependencies => investing in better hardware

We will analyze how to actually impliment register renaming and out-of-order execution

## Tomasulo's Algorithm

Goals:
1. Determine which instructions have inputs ready
2. Register renaming

Originally it was used for floating point instructions, now it's used for everything.

Parts:

Instruction Queue:
Instructions are fetched and put into a queue. FIFO, so they are in instruction order which allows renaming to work appropriately.

Reservation Station:
Instructions are **issued** from the queue to the reservation stations. Usually these have a limiting purpose (ADD/SUB, MUL/DIV). It is in this step that the renaming via the RAT happens, so that instructions can wait for results to be ready if they have true dependencies.

Registers:
When an instruction is put in the reservation station, if the values are ready, they are placed in the instruction in the reservation station.

Execution Unit:
Adder, Multiplier, etc. Instructions are **dispatched** from the associated reservation station. Computes the instruction then **broadcasts** via a BUS the result to the register file, the reservation stations and the RAT. This allows instructions that are waiting for their values to know that they are free to execute.

If the instruction is a load or store:
1. Goes to the address generation unit to compute the address
2. Insert into the load buffer or store buffer
3. Queued up to go into memory
4. When value comes back from memory, it is also broadcast on the bus.
5. Also the storebuffer is the recipiant of the broadcasts so that if it is true dependent it can get it's  values when they become available.

## Issue

- Take the next instruction (**in program order**) from the instruction queue.
- Determine where the inputs for the instruction come from (are they already in the register file or will they come from an instruction in one of the registration stations)
- Get a free reservation station (This must be of the correct kind and also have a free slot)
- Put instruction into the reservation station
- Update the RAT for the result value with the reservation station address

## Dispatch

- **Latch** produced broadcast results to insts waiting for specific RS result 
- Decide which inst is ready to execute (has all values, oldest first, longest executing first e.g. MUL/DIV)

## Broadcast

- Put tag (RS#) & result on Bus
- Write to Reg (use RAT to convert RS# -> F#)
- Update (clear) RAT
- Free RS

If one bus, the slower unit result is broadcast in priority. If stale result is broadcast (aka RAT doesn't contain RS# for result), it still latches to RS, but otherwise does nothing.

Per cycle:
- Issue
- Capture (latch)
- Dispatch
- Write Result

Same cycle:
Issue -> Dispatch? No
Capture -> Dispatch? No
Update RAT for Issue & Write Result? Yes but Issue updates RAT last

## Load and Store

In Tomasulo's done in order to avoid dependency problems
