---
title: "L1: Introduction, Metrics, and Evaluation"
date: 2020-03-02T20:00:47-05:00
---

# Introduction, Metrics, and Evaluation

## What is Computer Architecture

Design a computer that is well designed for it's purpose (e.g. the difference between a laptop, desktop, smartphone, server, etc). Laptop and smartphone would be focused on battery life, desktop on power, etc. We need computer architecture to improve performace (speed, battery life, size, weight) and abilities (3D graphics, security, debugging support).

Computer Architecture is the translation of hardware improvements into better performing and more capable software.

Must account for future technology and trends or else risk creating obsolete architecture. 
- Moore's law predicts that there are 2x transistors/area every 18-24 months. 
**the goal of computer architecture follows:**
- This means processor speed can double at that pace and serves as the metric to meet. 
- Power use/operation should half.
- Memory capacity should double

There is a gap between processor performance and latency. Memory latency only goes up 1.1x every two years. This leads to a "memory wall." Caches are the workaround for this. They allow for mitigation of the latency problem by drastically reducing the number of times the processor needs to pull from/store to memory (RAM).

There are tradeoffs to be weighed to obtain the improvements desired for the desired design. One system best utilizes 2x processing speed while cost and power consumption stay constant. Another might want a more moderate speed improvement while cost and power req are cut in half.

---

## Power Consumption

**Dynamic Power**

`P = 1/2 C * V^2 * f * a`
C is capacitance (chip area)
V is power supply voltage
f is frequency (chip Hz number)
a is activity factor (percent of transistors active in any clock cycle)

see the tradeoffs above for how the power consumption is a tradeoff game to keep power consumption the same/lower

**Static Power**

The lower the voltage the greater the leakage (exponentially). But the higher the voltage, the higher the dynamic power. There is a sweet spot in the middle where power consumption is lowest.

---

## Fabrication Cost

Chips are cut up 12" wafers that have gone through a costly production. There is a production yield which is the percentage of chips obtained out of a maximum potential. This yield is negatively affected by defects which take a static number away from the total number of chips produced. This greatly affects larger chips. In accordance to Moore's law we either get smaller chips for the same price or more speed for same price.
