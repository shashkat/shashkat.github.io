---
title: 'Notes about vectorization in CPUs'
date: 2025-12-13
permalink: posts/notes-about-vectorization-in-cpus/
tags:
  - Hardware
---

Some notes while thinking about vectorization and its connection with CPU clock rate

## Introduction

In the d2l book, in chapter 13 (Computational Performance) section 4 (Hardware), about vectorization it was mentioned:

"Deep learning is extremely compute-hungry. Hence, to make CPUs suitable for machine learning, one needs to perform many operations in one clock cycle. This is achieved via vector units."

So I began thinking about how does vectorization link with clock-rate of a CPU, and what exactly is one clock-cycle after all.

## Notes

- Firstly, in the CPU cores, there are separate units for dealing with vectorized operations. They are called SIMD (Single Input Multiple Data), and different types of them are used commonly for different types of architectures, like NEON for ARM and AVX2 for x86.

- In GPUs also there are separate vectorization units, but they are different from those in CPUs. They are smaller in size and spread across all the numerous cores in a GPU. They are generally able to perform vectorized operations on a much bigger tensor.

- In the X-TOY Machine (https://introcs.cs.princeton.edu/java/60machine/toy.pdf), the instructions were all in memory, and they were sweeped through one by one and executed. However, in modern CPUs, its slightly different. In the CPU, there is a register called program counter, which holds the memory address of the next instruction to be executed. That address is looked for in the caches hierarchically, and then if all were cache misses, in the RAM. Then, that information is fetched into another register in the CPU called instruction register (IR) and then it is decoded. Decoding here means interpreting the operation-code (bit segments that tell what to do like ADD, MOVE, JUMP etc.) and fields (bit segments that tell how to do it, like data locations, sub-operations if any etc.) from IR and setting up the control signals (make ready the different CPU components, like say ALU which will be involved in the execute step) for the execute stage which is next. The decode stage can take around 1-4 cycles, while the execute stage can take more too. These three steps (fetch-decode-execute) are repeated in the CPU continuously.

- The clock-rate of a CPU is the number of cycles of its CPU's clocking mechanism per second. To understand what one cycle is, a good starting point is to imagine one round of flipping of all transistor switches in the CPU, which had to be flipped in that step according to whatever instruction was being processed. Note that one step would not be computation of the full instruction, but one part of it (like a part of the execute step). More accurately, the cycles in CPU are like a metronome, which synchornizes the states of the transistors every cycle. So in simple terms, at the end of each cycle, the transistors reach the state they were meant to be in that step. In the middle of two cycles, maybe some of them were in their meant-to-be state while some were not. Hence, having more clock-rate is helpful as then the CPU synchronizes the transistors more often, and that means they the executions are being processed faster.

## References

- X-TOY Machine, which is a great visual tool to understand inner workings of a CPU: https://introcs.cs.princeton.edu/java/60machine/toy.pdf
- Useful answer from perplexity regarding which registers store instruction in CPU during the fetch-decode-execute process: https://www.perplexity.ai/search/when-cpu-is-processing-instruc-s0uma55YTwKw0E1uUtO_tQ#0
- Very useful video for understanding CPU at different levels: https://youtu.be/C7xLw0jNZU8?si=fjqfLoaAiwdlvtpp

