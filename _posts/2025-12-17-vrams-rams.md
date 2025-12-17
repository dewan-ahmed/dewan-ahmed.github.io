---
title: "VRAM vs System RAM: What Actually Limits Running LLMs Locally?"
last_modified_at: 2025-12-17T0:00:00-05:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/2025/vram-vs-systemram.png"
tags:
  - llm
  - ai
  - devrel
---

If you’ve spent any time experimenting with running large language models locally, you’ve probably seen guidance like _"you need 30GB of RAM"_ to get a model working. That exact phrasing showed up recently on a [Reddit thread](https://www.reddit.com/r/LocalLLM/comments/1p8xlnw/run_qwen3next_locally_guide_30gb_ram/) about running **Qwen3-Next locally**, and it sparked a familiar kind of confusion: _are we talking about GPU memory or system memory?_

The short answer is that both matter—but in very different ways. Understanding the distinction between **VRAM** and **system RAM** is one of the most important mental models for anyone building or tuning local LLM inference. Once you get it, a lot of vague "hardware requirements" suddenly make sense.

## Two memory systems, two very different jobs

When you run an LLM locally, you are operating across at least two memory hierarchies: **GPU memory (VRAM)** and **system memory (RAM)**. They serve different purposes and impose different limits on what’s possible.

### VRAM: where inference actually happens

VRAM is the memory physically attached to your GPU. During GPU-based inference, this is where the performance-critical parts of the model live:

- Model weights in their inference format (FP16, INT8, INT4, etc.)
- Activation buffers used during forward passes
- The KV cache that stores attention state for the active context
- Temporary buffers used by fused kernels and matrix multiplications

Modern inference runtimes (vLLM, llama.cpp, TensorRT-LLM, and PyTorch-based stacks) try very hard to keep as much of this as possible in VRAM. The reason is simple: GPU memory offers dramatically higher bandwidth and better effective latency for GPU workloads than system RAM.

If the full model (plus its KV cache and runtime buffers) fits in VRAM, you get fast, predictable throughput. If it doesn’t, you’re forced into offloading strategies where parts of the model or attention state live in system RAM. That works, but performance drops sharply as data moves back and forth across the CPU–GPU boundary.

In practice, **VRAM is the gating factor for model size on GPU**.

### System RAM: the support structure

System RAM plays a quieter but still critical role. It doesn’t directly accelerate inference kernels, but it supports everything around them:

- Holding unpacked model files during load
- Staging unquantized weights before conversion
- Temporary buffers used during quantization
- Offloaded layers or tensors that don’t fit in VRAM
- CPU-only inference paths when no GPU is available

If system RAM is insufficient, models may fail to load entirely or spend most of their time paging data between memory and disk. This is often where seemingly large RAM requirements come from.

In the [Qwen3-Next discussion](https://www.reddit.com/r/LocalLLM/comments/1p8xlnw/run_qwen3next_locally_guide_30gb_ram/), the frequently cited _"30GB of RAM"_ wasn’t referring to VRAM at all—it was **system RAM**. Even if the final quantized model fits comfortably on the GPU, the loader and runtime may still need substantial host memory to hold intermediate representations during setup.

## Why VRAM constraints show up first

For GPU inference, VRAM determines feasibility before performance even enters the conversation.

If a model’s weights and KV cache cannot reside in GPU memory at the same time, you have three options:

1. Use more aggressive quantization
2. Reduce context length (to shrink the KV cache)
3. Accept CPU offloading and slower inference

For large models (30B parameters and above), this usually means **24GB of VRAM or more** just to get started, and significantly more if you want higher precision or long-context workloads. Context length matters here: KV cache memory grows linearly with the number of tokens, and for long prompts it can rival or exceed the size of the model weights themselves.

This is why two users running the "same model" can have wildly different experiences depending on context length and precision.

## System RAM enables flexibility, not speed

System RAM doesn’t make inference faster, but it makes it _possible_.

Adequate RAM allows:

- Safe model loading and conversion
- Partial offload paths when VRAM is tight
- CPU-only inference for users without GPUs

Without it, runtimes thrash, allocations fail, and performance collapses. That’s why RAM recommendations often sound inflated—they’re accounting for worst-case loading and conversion paths, not steady-state inference.

## CPU-only inference: feasible, but slower

Some inference engines can run entirely from system RAM with no GPU at all, using multithreaded CPU execution. This can be surprisingly usable for smaller or aggressively quantized models, where users report tens of tokens per second on modern CPUs.

For larger models, though, throughput quickly drops into single-digit tokens per second. CPU-only inference trades raw speed for accessibility, not efficiency.

## Thinking in memory hierarchies

A useful way to reason about local LLM hardware is as a memory hierarchy:

- **VRAM** → model weights, activations, KV cache
- **System RAM** → runtime support, offload, conversion
- **Storage** → model files, checkpoints, swap

When these layers are aligned with your inference goals, hardware choices become much clearer. Instead of chasing vague "requirements", you can decide whether you’re optimizing for feasibility, performance, context length, or cost.

## One last thing

I’m also writing these posts as part of my own transition from DevOps into AI and ML. Earlier in my career, I went through a similar shift when I moved from being a Java developer into DevOps, and writing things down publicly helped me make sense of that transition.

This blog is my way of doing the same thing again. Learning in public, validating my understanding, and slowly building intuition around systems that are still new to me. If you’re on a similar journey, I hope these notes help you connect a few dots along the way.
