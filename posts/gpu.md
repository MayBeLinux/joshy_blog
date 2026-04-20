# How GPUs Actually Work

*Published March 28, 2026 · 12 min read*

You've heard GPUs are good at parallel computation. You know they're used for gaming, machine learning, and crypto. But *why* are they fast? What's actually happening inside? Let's go deep.

## The CPU vs GPU Mental Model

A CPU is designed to execute a **single thread as fast as possible**. It has a few powerful cores (4–32), large caches, sophisticated branch predictors, and out-of-order execution engines — all to minimize latency on sequential tasks.

A GPU does the opposite. It has **thousands of smaller, simpler cores** designed to execute many threads simultaneously. It trades latency for throughput. A single GPU operation might take longer than a CPU equivalent, but it can do tens of thousands of them *at the same time*.

| | CPU | GPU |
|---|---|---|
| Cores | 4–32 powerful | Thousands of small |
| Optimization target | Single-thread latency | Total throughput |
| Cache | Large per-core | Smaller, shared |
| Best for | Sequential logic | Parallel data |

## SIMT: Single Instruction, Multiple Threads

The fundamental execution model of a GPU is **SIMT**. A group of threads (called a **warp** on NVIDIA, a **wavefront** on AMD) execute the *same instruction* simultaneously, but on *different data*.

Imagine you need to multiply 10,000 pairs of numbers. A CPU does them sequentially. A GPU groups 32 threads into a warp, each thread gets its own pair, and the multiply instruction fires on all 32 simultaneously — one clock cycle, 32 results.

```
Thread 0: 3.4 × 2.1 = 7.14
Thread 1: 1.2 × 5.6 = 6.72
Thread 2: 8.0 × 0.9 = 7.20
... (32 threads, one instruction)
```

## Shader Cores and SMs

NVIDIA organizes its GPU into **Streaming Multiprocessors (SMs)**. Each SM contains:

- Multiple **CUDA cores** (FP32 units)
- Tensor Cores (matrix math, used for ML)
- RT Cores (ray tracing)
- Shared memory and L1 cache
- A warp scheduler

A modern high-end GPU like the RTX 4090 has **128 SMs**, each with 128 CUDA cores — that's **16,384 cores total**. The warp scheduler inside each SM keeps those cores fed by switching between warps when one is waiting on memory.

## Memory Hierarchy

This is where GPU performance gets nuanced. Memory latency can kill throughput if you're not careful.

```
Registers       → fastest, per-thread, ~1 cycle
Shared memory   → fast, per-SM, ~5 cycles (programmer-managed L1)
L2 cache        → shared across SMs, ~200 cycles
VRAM (GDDR/HBM) → slowest, ~600 cycles but high bandwidth
```

The key insight: **bandwidth > latency for GPUs**. An RTX 4090 has ~1 TB/s of memory bandwidth. The strategy is to keep data in shared memory and registers, batch your memory accesses, and let the warp scheduler hide latency by switching to other warps while memory fetches complete.

## Branch Divergence

SIMT's weakness: branches. If threads in a warp take different paths in an `if/else`, the GPU must execute **both branches serially**, masking out the threads that don't take each path. Half your throughput, gone.

```c
// Bad for GPU — half the warp idles in each branch
if (threadId % 2 == 0) {
    doSomething();
} else {
    doSomethingElse();
}
```

Good GPU code minimizes divergence. This is why GPU algorithms often look strange to CPU programmers — they're designed around this constraint.

## Why GPUs Excel at Machine Learning

Matrix multiplication is at the heart of neural networks. Multiplying two 1024×1024 matrices requires ~2 billion multiply-add operations. This is *exactly* the kind of regular, branch-free, massively parallel work GPUs are designed for.

NVIDIA's Tensor Cores take this further — they're specialized hardware units that perform 4×4 matrix multiply-accumulate in a single operation, giving another 8–16× throughput boost over regular CUDA cores for FP16/BF16 math.

## The Takeaway

A GPU is not a "fast CPU." It's a different computational paradigm:

- **Thousands of simple cores** instead of dozens of complex ones
- **SIMT execution** — same instruction, many threads, much data
- **Latency hiding** through massive parallelism and warp switching
- **Bandwidth-optimized** memory with a programmer-managed hierarchy

When your problem maps onto this model — graphics rendering, ML training, physics simulation, signal processing — a GPU is orders of magnitude faster than a CPU. When it doesn't — dynamic branching, pointer chasing, sequential logic — the CPU wins easily.

Understanding this boundary is what separates programmers who *use* GPUs from those who truly *program* them.
