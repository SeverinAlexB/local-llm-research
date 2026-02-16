# Apple Silicon Path — Research Report

> Researched: February 2026 | Purchase timing: February 2026

---

## Table of Contents

1. [Current Mac Lineup for AI Inference](#1-current-mac-lineup-for-ai-inference)
2. [Mac Studio M3 Ultra — Detailed Configurations & Swiss Pricing](#2-mac-studio-m3-ultra)
3. [Mac Studio M4 Max — Detailed Configurations & Swiss Pricing](#3-mac-studio-m4-max)
4. [Mac Pro M2 Ultra — Current Status](#4-mac-pro-m2-ultra)
5. [Performance Benchmarks](#5-performance-benchmarks)
6. [Concurrent Inference & Multi-Agent Feasibility](#6-concurrent-inference--multi-agent-feasibility)
7. [Software Ecosystem & Limitations](#7-software-ecosystem--limitations)
8. [Refurbished Availability](#8-refurbished-availability)
9. [Upcoming Hardware — M5 Ultra Timeline](#9-upcoming-hardware--m5-ultra-timeline)
10. [Recommendations Against Requirements](#10-recommendations-against-requirements)

---

## 1. Current Mac Lineup for AI Inference

**Critical finding**: There is no M4 Ultra chip as of February 2026. Apple's 2025 Mac Studio refresh paired the M4 Max with the M3 Ultra (previous generation). Apple told reviewers that "not every generation of M-series chips will include a higher-end Ultra tier." Bloomberg's Mark Gurman reported in November 2025 that Apple has "largely written off the Mac Pro" and considers the Mac Studio its high-end workstation solution.

The relevant options today are:

| Machine | Chip | Max Memory | Memory Bandwidth | Swiss Base Price |
|---|---|---|---|---|
| Mac Studio (2025) | M4 Max (16C CPU / 40C GPU) | 128 GB | 546 GB/s | CHF 2,599 |
| Mac Studio (2025) | M3 Ultra (28C CPU / 60C GPU) | 256 GB (BTO) | 819 GB/s | CHF 4,199 |
| Mac Studio (2025) | M3 Ultra (32C CPU / 80C GPU) | 512 GB (BTO) | 819 GB/s | CHF 5,699 |
| Mac Pro (2023) | M2 Ultra (24C CPU / 60C GPU) | 192 GB | 800 GB/s | CHF 7,199 |
| Mac Pro (2023) | M2 Ultra (24C CPU / 76C GPU) | 192 GB | 800 GB/s | CHF 8,299 |

Sources: [EveryMac Switzerland Prices](https://everymac.com/global-mac-prices/all-msrp-mac-studio-prices-in-switzerland.html), [Apple Mac Studio Specs](https://www.apple.com/mac-studio/specs/), [Apple CH Store](https://www.apple.com/ch-de/shop/buy-mac/mac-studio)

---

## 2. Mac Studio M3 Ultra

### Specifications

| Spec | 28C/60C GPU (Base) | 32C/80C GPU (High) |
|---|---|---|
| CPU Cores | 28 (20P + 8E) | 32 (24P + 8E) |
| GPU Cores | 60 | 80 |
| Neural Engine | 32-core | 32-core |
| Memory Options | 96 GB, 256 GB | 96 GB, 256 GB, 512 GB |
| Memory Bandwidth | 819 GB/s | 819 GB/s |
| Storage Options | 1-8 TB (base) / 1-16 TB (high) | 1-16 TB |

### Swiss Pricing (CHF, Apple Store)

**Base configurations (from EveryMac/Apple CH):**

| Configuration | CHF (estimated) |
|---|---|
| M3 Ultra 28C/60C, 96 GB, 1 TB | CHF 4,199 |
| M3 Ultra 32C/80C, 96 GB, 1 TB | CHF 5,699 |

**Memory upgrade costs (from US pricing, ~1:1.1 USD:CHF ratio):**

| Upgrade | USD Cost | Estimated CHF |
|---|---|---|
| 96 GB -> 256 GB (on 28C/60C) | +$1,600 | ~+CHF 1,700 |
| 96 GB -> 256 GB (on 32C/80C) | +$1,500 | ~+CHF 1,600 |
| 96 GB -> 512 GB (on 32C/80C) | +$2,400 | ~+CHF 2,500 |

**Key configurations for your use case:**

| Configuration | Estimated CHF |
|---|---|
| M3 Ultra 28C/60C, **256 GB**, 1 TB | **~CHF 5,900** |
| M3 Ultra 28C/60C, **256 GB**, 2 TB | **~CHF 6,100** |
| M3 Ultra 32C/80C, **256 GB**, 2 TB | **~CHF 7,500** |
| M3 Ultra 32C/80C, **512 GB**, 2 TB | **~CHF 8,400** |
| Maxed out (32C/80C, 512 GB, 16 TB) | **~CHF 14,300** |

**The sweet spot for your requirements: M3 Ultra 28C/60C with 256 GB, at approximately CHF 5,900-6,100.** This fits comfortably in the budget sweet spot of CHF 5,000-12,000.

Sources: [MacRumors - Maxed Out Pricing](https://www.macrumors.com/2025/03/05/maxed-out-m3-ultra-mac-studio-14099/), [Apple US Store - 256GB Config](https://www.apple.com/shop/buy-mac/mac-studio/m3-ultra-chip-28-core-cpu-60-core-gpu-256gb-memory-1tb-storage)

---

## 3. Mac Studio M4 Max

### Specifications

| Spec | 14C/32C GPU (Base) | 16C/40C GPU (High) |
|---|---|---|
| CPU Cores | 14 (10P + 4E) | 16 (12P + 4E) |
| GPU Cores | 32 | 40 |
| Neural Engine | 16-core | 16-core |
| Memory Options | 36 GB, 48 GB, 64 GB | 36 GB, 48 GB, 64 GB, 128 GB |
| Memory Bandwidth | 410 GB/s | 546 GB/s |
| Storage Options | 512 GB - 8 TB | 512 GB - 8 TB |

### Swiss Pricing (CHF)

| Configuration | CHF |
|---|---|
| M4 Max 14C/32C, 36 GB, 512 GB | CHF 2,099 |
| M4 Max 16C/40C, 64 GB, 1 TB | CHF 2,599 |
| M4 Max 16C/40C, **128 GB**, 1 TB | ~CHF 3,200 (estimated with upgrade) |

### Why M4 Max is insufficient for your requirements

- **Maximum 128 GB** unified memory — below your 256 GB target and only meets the minimum
- **546 GB/s bandwidth** vs M3 Ultra's 819 GB/s — 33% less throughput for memory-bound LLM inference
- A 70B Q4 model takes ~40 GB, a 70B Q8 takes ~75 GB. With 128 GB, you can run a 70B Q4 + a 32B model, but it leaves very little headroom for concurrent vision models or multiple sub-agent models
- At higher quantizations (Q8, FP8), 128 GB becomes extremely tight

Sources: [Apple Mac Studio Specs](https://www.apple.com/mac-studio/specs/), [Apple Support Specs](https://support.apple.com/en-us/122211)

---

## 4. Mac Pro M2 Ultra

### Current Status

The Mac Pro was last updated in 2023 with the M2 Ultra. Bloomberg's Gurman reported Apple has "largely written off the Mac Pro." No M4 Ultra Mac Pro appears to be coming.

### Specifications & Swiss Pricing

| Configuration | Memory | Bandwidth | CHF |
|---|---|---|---|
| M2 Ultra 24C/60C, Tower | up to 192 GB | 800 GB/s | CHF 7,199 |
| M2 Ultra 24C/76C, Tower | up to 192 GB | 800 GB/s | CHF 8,299 |
| M2 Ultra 24C/60C, Rack | up to 192 GB | 800 GB/s | CHF 7,799 |
| M2 Ultra 24C/76C, Rack | up to 192 GB | 800 GB/s | CHF 8,899 |

### Verdict: Not recommended

- **Max 192 GB** — still does not reach 256 GB target
- **800 GB/s** bandwidth vs M3 Ultra's 819 GB/s — slightly slower
- **Older chip** (M2 generation) — worse compute per watt and per core
- **More expensive** than M3 Ultra Mac Studio for less capability
- Mac Pro's PCIe expansion slots are wasted since Apple Silicon uses unified memory (you cannot add discrete GPUs)
- The Mac Pro's only remaining advantage is internal storage expansion, which is irrelevant for inference

Sources: [EveryMac Mac Pro Prices CH](https://everymac.com/global-mac-prices/all-msrp-mac-pro-prices-in-switzerland.html), [Macworld Mac Pro Rumors](https://www.macworld.com/article/2320613/new-mac-pro-ultra-release-date-specs-price-m4-m5.html)

---

## 5. Performance Benchmarks

### 5.1 Token Generation — 70B Models

These are **generation (output) speeds**, the metric that matters most for interactive coding.

| Hardware | Model | Quant | Gen tok/s | Source |
|---|---|---|---|---|
| M3 Ultra 256GB | DeepSeek R1 70B | Q4 | **13.7 tok/s** | [Hostbor](https://hostbor.com/mac-studio-m3-ultra-tested/) |
| M3 Ultra | Llama 3 70B | Q4_K_M | **12-18 tok/s** | [Lattice Benchmarks](https://lattice.uptownhr.com/local-llm-inference/m3-ultra-performance-benchmarks) |
| M3 Ultra | Llama 3 70B | 4-bit (Metal) | **~15 tok/s** | [Lattice](https://lattice.uptownhr.com/local-llm-inference/m3-ultra-performance-benchmarks) |
| M4 Max 128GB | DeepSeek R1 70B | Q4 | **8.8 tok/s** | [Hostbor](https://hostbor.com/mac-studio-m3-ultra-tested/) |
| M4 Max 128GB | 70B models (MLX) | Q4 | **30-45 tok/s** | [InsiderLLM](https://www.insiderllm.com/guides/best-local-llms-mac-2026/) |
| M2 Ultra 192GB | DeepSeek R1 70B | Q4 | **8.0 tok/s** | [Hostbor](https://hostbor.com/mac-studio-m3-ultra-tested/) |
| M2 Ultra 192GB | Llama 3 70B | Q4_K_M | **~12 tok/s** | [llama.cpp discussion](https://github.com/ggml-org/llama.cpp/discussions/4167) |

**Note on variance**: The wide range (8-45 tok/s) for 70B models on M4 Max reflects different models, quantization levels, context lengths, and frameworks. DeepSeek R1 70B is a particularly demanding model. Llama 3.3 70B at Q4 with MLX on M4 Max reportedly achieves ~30-40 tok/s, while the same model through llama.cpp may only get ~7-15 tok/s.

### 5.2 Token Generation — Small/Medium Models

| Hardware | Model | Quant | Gen tok/s | Source |
|---|---|---|---|---|
| M3 Ultra 256GB | Llama 8B | 4-bit | **128 tok/s** | [Creative Strategies](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/) |
| M3 Ultra 256GB | IBM Granite 8B | 4-bit | **108 tok/s** | [Creative Strategies](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/) |
| M3 Ultra | Gemma 3 4B | Q4 | **134 tok/s** | [Lattice](https://lattice.uptownhr.com/local-llm-inference/m3-ultra-performance-benchmarks) |
| M3 Ultra | Gemma 3 1B | Q4 | **237 tok/s** | [Lattice](https://lattice.uptownhr.com/local-llm-inference/m3-ultra-performance-benchmarks) |
| M3 Ultra 256GB | Phi-4 14B | 4-bit | **72 tok/s** | [Creative Strategies](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/) |
| M3 Ultra 256GB | Gemma 2 9B | 4-bit | **82 tok/s** | [Creative Strategies](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/) |
| M3 Ultra 256GB | QwQ 32B | 4-bit | **33-37 tok/s** | [Creative Strategies](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/) |
| M3 Ultra | Qwen2.5-Coder 32B | 4-bit | **~18-20 tok/s** | [Lattice](https://lattice.uptownhr.com/local-llm-inference/m3-ultra-performance-benchmarks) |
| M3 Ultra | DeepSeek R1 (full 671B) | 4-bit | **16-18 tok/s** | [Hostbor](https://hostbor.com/mac-studio-m3-ultra-tested/) |

### 5.3 Prompt Processing (Prefill) Speed

| Hardware | Model (7B) | Quant | Prefill tok/s |
|---|---|---|---|
| M3 Ultra (80C GPU) | LLaMA 7B | F16 | 1,538 tok/s |
| M3 Ultra (80C GPU) | LLaMA 7B | Q4_0 | 1,471 tok/s |
| M4 Max (32C GPU) | LLaMA 7B | F16 | 748 tok/s |
| M4 Max (32C GPU) | LLaMA 7B | Q4_0 | 716 tok/s |
| M2 Ultra (76C GPU) | LLaMA 7B | F16 | 1,402 tok/s |

Source: [llama.cpp Apple Silicon Discussion](https://github.com/ggml-org/llama.cpp/discussions/4167)

### 5.4 MLX vs llama.cpp Performance

MLX (Apple's native ML framework) consistently outperforms llama.cpp on Apple Silicon:

- **20-30% faster** across model sizes, with the gap widening on larger models
- MLX uses lazy evaluation and fused operations optimized for the unified memory architecture
- For serving, **vllm-mlx** achieves 21-87% higher throughput than llama.cpp across tested models

Source: [Production-Grade LLM Inference Paper](https://arxiv.org/abs/2511.05502), [vllm-mlx Paper](https://arxiv.org/html/2601.19139v1)

### 5.5 Concurrent Scaling (Olares Blog Benchmarks)

Tested on Mac Studio M3 Ultra with the Olares benchmark suite:

| Model | 1 concurrent | 8 concurrent |
|---|---|---|
| Qwen3-30B-A3B | 84 tok/s | 24.9 tok/s |
| GPT-OSS-120B | 69.4 tok/s | 19.1 tok/s |

**Note:** The Olares blog also benchmarked an "Olares One" device (GPU-based, running vLLM) which achieved 157 tok/s for Qwen3-30B-A3B. That figure applies to the Olares One, not the Mac Studio.

On M4 Max:

| Model | 1 concurrent | 8 concurrent |
|---|---|---|
| Qwen3-30B-A3B | 81.2 tok/s | 19.9 tok/s |

Source: [Olares Blog - Hardware Performance Benchmarking](https://blog.olares.com/local-ai-hardware-performance-benchmarking/)

---

## 6. Concurrent Inference & Multi-Agent Feasibility

### The Concurrency Challenge

This is the most critical concern for your multi-agent use case.

**MLX (raw framework):**
- MLX crashes when attempting concurrent inference of independent models from separate threads due to thread-safety issues in the Metal backend ([GitHub Issue #3078](https://github.com/ml-explore/mlx/issues/3078))
- Single-stream optimized; no built-in scheduler or batching
- Workaround: Run multiple separate MLX processes behind a load balancer

**vllm-mlx (recommended for your use case):**
- Provides continuous batching on Apple Silicon
- Scales to **4.3x aggregate throughput** at 16 concurrent requests
- Qwen3-0.6B: 441 tok/s (1 req) -> 1,642 tok/s (16 concurrent) = 3.7x scaling
- Qwen3-8B: 2.6x scaling at 16 concurrent requests
- Supports OpenAI-compatible API, MCP tool calling, and multimodal (vision) models
- Source: [vllm-mlx Paper](https://arxiv.org/html/2601.19139v1)

**MLC-LLM:**
- Best option for multi-worker HTTP serving with micro-batching
- Reduces head-of-line blocking for concurrent interactive workloads
- Source: [Production-Grade LLM Inference Paper](https://arxiv.org/abs/2511.05502)

### Practical Assessment for Your Requirements

With an M3 Ultra 256 GB running your target workload (1x 70B main + multiple 7B-32B sub-agents + vision):

**Memory budget (256 GB):**

| Model | Quant | Approx. Memory |
|---|---|---|
| 70B coding model (main agent) | Q4 | ~40 GB |
| 70B coding model (main agent) | Q8/FP8 | ~75 GB |
| 32B sub-agent model | Q4 | ~18 GB |
| 7B sub-agent model | Q4 | ~4 GB |
| Vision model (e.g., Qwen2-VL 7B) | Q4 | ~5 GB |
| KV cache overhead (multiple streams) | — | ~20-40 GB |
| OS + system overhead | — | ~8-16 GB |
| **Total (Q4 main agent)** | | **~95-125 GB** |
| **Total (FP8 main agent)** | | **~130-160 GB** |

With 256 GB, you have significant headroom in Q4 mode or comfortable fit in FP8 mode.

**Throughput estimate (M3 Ultra 256 GB):**

| Stream | Model | Expected tok/s |
|---|---|---|
| Main agent (1x) | 70B Q4 via MLX | 15-25 tok/s single-stream |
| Sub-agents (5x) | 32B Q4 via vllm-mlx batched | ~15-20 tok/s each (batched) |
| Vision model (1x) | 7B VL model | Concurrent via separate process |

**Honest assessment**: With **dense 70B models**, the main agent at 15-25 tok/s falls short of the 60-80 tok/s target. However, with **MoE models** (the industry standard for frontier coding models), GPT-OSS-120B achieves **~69 tok/s** on M3 Ultra — meeting the target in single-stream mode. Under heavy concurrency (8 streams), it drops to ~19 tok/s. Sub-agent MoE models (Qwen3-30B-A3B) at ~84 tok/s single / ~25 tok/s concurrent easily meet their targets. The main tradeoff vs NVIDIA RTX PRO 6000 Blackwell: the PRO 6000 achieves ~134 tok/s on the same MoE models (nearly 2x faster) and degrades less under concurrent load.

---

## 7. Software Ecosystem & Limitations

### What Works Well

- **MLX**: Apple's native ML framework, 20-30% faster than llama.cpp for inference
- **llama.cpp (Metal backend)**: Well-supported, broad model compatibility
- **Ollama**: Easy to use, Metal-accelerated, good for quick deployment
- **vllm-mlx**: OpenAI-compatible API with continuous batching, MCP tool calling, multimodal support
- **LM Studio**: GUI-based, Metal-accelerated
- **PyTorch MPS backend**: Accelerated training/inference via Metal Performance Shaders
- **Headless operation**: Mac Studio works well as a headless inference server via SSH/API

### What Does NOT Work / Limitations

| Category | Issue | Impact |
|---|---|---|
| **CUDA** | Not available on macOS at all | Many AI tools assume CUDA; need Metal/MLX alternatives |
| **ROCm** | Not supported on macOS | AMD GPU compute stack is Linux-only |
| **vLLM (core)** | Native vLLM requires CUDA/ROCm | Must use vllm-mlx plugin (community-maintained) |
| **Multi-GPU** | Apple Silicon has no multi-GPU support | Cannot combine two Mac Studios for more VRAM |
| **Training/Fine-tuning** | PyTorch MPS has operator gaps and bugs | Some training ops silently fail or fall back to CPU; CUDA is far more mature for training |
| **Distributed training** | Not supported on MPS | Cannot do multi-node training |
| **Image generation** | 2-8x slower than NVIDIA RTX 4090/5090 | Flux.1 runs ~8x slower on M3 Ultra vs RTX 5090 |
| **SDPA bugs** | scaled_dot_product_attention crashes on MPS | Active PyTorch bug as of 2025 |
| **Concurrent MLX threads** | MLX Metal backend not thread-safe | Must use separate processes, not threads |
| **TensorRT** | NVIDIA-only optimization toolkit | No equivalent optimization on Apple Silicon |
| **Docker GPU passthrough** | Limited MPS support in containers | GPU acceleration in Docker is not as seamless as NVIDIA Container Toolkit |

### Image Generation Specifically

- Apple Silicon can run Stable Diffusion (SD1.5, SDXL) and Flux via MLX or MPS
- Performance is **2-8x slower** than comparable NVIDIA GPUs
- CoreML can leverage the Neural Engine for some image tasks
- For your use case (image recognition, not generation), this is less of a concern — vision models for screenshot analysis run fine via MLX/vllm-mlx

Sources: [Apple Silicon vs NVIDIA CUDA Comparison](https://scalastic.io/en/apple-silicon-vs-nvidia-cuda-ai-2025/), [Flux Apple Silicon Guide](https://apatero.com/blog/flux-apple-silicon-m1-m2-m3-m4-complete-performance-guide-2025)

---

## 8. Refurbished Availability

### Apple Refurbished Store Switzerland (apple.com/ch-de/shop/refurbished)

As of February 2026, the Swiss refurbished store shows limited Mac Studio inventory:

- Mac Studio (2023) M2 Max 64 GB, 1 TB — ~CHF 1,099
- Mac Studio (2023) M2 Max 128 GB, 4 TB — price varies

**No M3 Ultra or M2 Ultra Mac Studio units** were found in the refurbished store. This is expected since the M3 Ultra Mac Studio only launched in March 2025 — refurbished units typically appear 6-12 months after launch.

**No Mac Pro refurbished units** were found in the Swiss store.

### Third-Party Refurbished

- **Revendo.ch** lists some Mac Studio configurations but primarily newer/used models at near-retail prices
- The 2025 Mac Studio is too new for meaningful refurbished discounts

**Recommendation**: For the M3 Ultra, buy new from Apple CH. The refurbished market does not have relevant configurations available yet.

Sources: [Apple CH Refurbished](https://www.apple.com/ch-de/shop/refurbished/mac/mac-studio), [Refurb Tracker](https://refurb-tracker.com/feeds/cx_in_all.html)

---

## 9. Upcoming Hardware — M5 Ultra Timeline

Bloomberg's Mark Gurman reported (November 2025) that Apple has M5 Max and M5 Ultra Mac Studio on its 2026 release schedule.

| Expected | Hardware | Likely Specs |
|---|---|---|
| Spring 2026 (March-April) | Mac Studio M5 Max | ~128 GB max, improved bandwidth |
| Summer-Fall 2026 (June-Sept) | Mac Studio M5 Ultra | ~512 GB max, likely >900 GB/s bandwidth |

**Should you wait?**
- The M5 Max Mac Studio could arrive as early as March 2026 but will still cap at ~128 GB (insufficient)
- The M5 Ultra Mac Studio is the one that matters, but it is **4-7 months away** (June-September 2026)
- If you can wait until fall 2026, M5 Ultra will likely offer significantly better performance at the same bandwidth tier or better
- If you need hardware now (February 2026), the M3 Ultra is the only Apple option that meets the 256 GB requirement

Sources: [MacRumors M5 Ultra](https://www.macrumors.com/2025/11/04/mac-studio-m5-ultra-2026/), [Macworld 2026 Mac Studio](https://www.macworld.com/article/2973459/2026-mac-studio-what-we-know-about-the-upcoming-m5-update.html), [AppleInsider M5 Mac Studio](https://appleinsider.com/articles/26/02/08/m5-max-mac-studio-new-studio-display-could-finally-arrive-in-the-spring)

---

## 10. Recommendations Against Requirements

### Requirement Fit Matrix

| Requirement | Target | M3 Ultra 256GB | M4 Max 128GB | Mac Pro M2 Ultra 192GB |
|---|---|---|---|---|
| GPU-accessible memory | 96-256 GB | 256 GB | 128 GB (min only) | 192 GB (partial) |
| Main agent speed (MoE) | 60-80 tok/s | **~69 tok/s (PASS)** — GPT-OSS-120B | ~35 tok/s (PARTIAL) | ~45-55 tok/s (est., PARTIAL) |
| Main agent speed (dense 70B) | 30 tok/s min | 15-25 tok/s (MISS) | 8-15 tok/s (MISS) | 10-15 tok/s (MISS) |
| Sub-agent speed (MoE) | 15-20 tok/s | **~84 tok/s single / ~25 tok/s at 8 concurrent (PASS)** | ~81 tok/s single (PASS) | ~100+ tok/s (est., PASS) |
| Concurrent streams | 5-10 | Feasible with vllm-mlx | Very tight on memory | Feasible but less memory |
| Image recognition | Concurrent | PASS | PASS (tight) | PASS |
| Noise | Quiet home office | Silent (no fan at idle) | Silent | Louder (tower fans) |
| Budget | 4,000-15,000 CHF | ~CHF 5,900-6,100 | ~CHF 3,200 | CHF 7,199-8,299 |
| Headless server | SSH/API | PASS | PASS | PASS |

**MoE note:** The industry has shifted to MoE (Mixture of Experts) coding models that only activate a fraction of parameters per token. This dramatically changes the Apple Silicon story — GPT-OSS-120B (120B total, ~20B active) achieves ~69 tok/s on M3 Ultra, meeting the 60-80 tok/s target. See Section 5.5 for benchmarks.

### MoE Models Change the Picture

With **dense 70B models**, no Apple Silicon can hit 60-80 tok/s — the M3 Ultra achieves only ~15-25 tok/s on 70B Q4. However, the industry has shifted to **MoE (Mixture of Experts)** coding models, which only activate a fraction of parameters per token. This changes everything:

| Model | Type | M3 Ultra tok/s (single) | Meets 60-80 target? |
|---|---|---|---|
| GPT-OSS-120B | MoE (120B/~20B active) | **~69 tok/s** | **Yes** |
| Qwen3-30B-A3B | MoE (30B/3.3B active) | **~84 tok/s** | Far exceeds |
| Qwen2.5-Coder 72B | Dense (72B) | ~15-20 tok/s | No |

**The practical strategy on Apple Silicon**: Use **GPT-OSS-120B** (MoE) as the main coding agent at ~69 tok/s, and **Qwen3-Coder-30B-A3B** (MoE) for sub-agents at ~84 tok/s. Fall back to dense 70B models only for tasks where MoE models underperform.

**Caveat**: Under heavy concurrency (8 streams), MoE performance drops significantly — GPT-OSS-120B falls from ~69 to ~19 tok/s. The NVIDIA RTX PRO 6000 Blackwell handles concurrency better (134 tok/s single, ~80-100 tok/s under load).

### Best Apple Silicon Configuration

**Recommended: Mac Studio M3 Ultra 28C/60C, 256 GB, 2 TB**
- Estimated price: **~CHF 6,100**
- Meets: memory target (256 GB), MoE main agent speed (~69 tok/s), sub-agent speed, concurrent inference, noise, budget
- Falls short: dense 70B speed (15-25 tok/s), concurrent MoE performance degrades faster than NVIDIA
- Excellent for: running multiple models simultaneously, silent operation, macOS ecosystem integration, long-term value
- **MoE strategy**: Use GPT-OSS-120B as main agent (~69 tok/s), Qwen3-Coder-30B-A3B for sub-agents (~84 tok/s)

### Price-to-Performance Summary

| Config | CHF | Memory | 70B tok/s | Notes |
|---|---|---|---|---|
| M4 Max 128GB | ~3,200 | 128 GB | 8-15 | Budget option; memory-limited |
| M3 Ultra 256GB (28/60) | ~5,900 | 256 GB | 15-25 | Best value for 256GB target |
| M3 Ultra 256GB (32/80) | ~7,500 | 256 GB | 15-25 | +33% GPU cores, marginal LLM gain |
| M3 Ultra 512GB (32/80) | ~8,400 | 512 GB | 15-25 | Overkill for current needs; future-proof |
| Mac Pro M2 Ultra 192GB | ~7,200 | 192 GB | 10-15 | Overpriced, outdated, dead product line |

---

## Key Takeaways

1. **The M3 Ultra 256 GB Mac Studio at ~CHF 5,900 is the clear Apple Silicon choice** — it is the only current Mac that reaches 256 GB GPU-accessible memory within budget.

2. **With MoE models, 60-80 tok/s IS achievable on Apple Silicon.** GPT-OSS-120B (MoE) runs at ~69 tok/s on M3 Ultra — within the target range. Dense 70B models still only get ~15-25 tok/s. The practical strategy is to use MoE coding models for speed-critical tasks. However, the NVIDIA RTX PRO 6000 Blackwell achieves ~134 tok/s on the same MoE models — nearly 2x faster — and handles concurrency better.

3. **Concurrent multi-agent inference is feasible** but requires vllm-mlx or MLC-LLM rather than raw MLX. Memory bandwidth (819 GB/s) is shared across all concurrent streams, so throughput per stream degrades with concurrency (84 tok/s single → 25 tok/s at 8 concurrent for Qwen3-30B-A3B; 69 tok/s single → 19 tok/s at 8 concurrent for GPT-OSS-120B).

4. **Do not buy the Mac Pro.** It is older, more expensive, louder, capped at 192 GB, and Apple has abandoned the product line.

5. **If you can wait until fall 2026**, the M5 Ultra Mac Studio will likely offer significantly better performance (potentially 30-40% faster inference) with the same or larger memory options. The M5 Max is coming sooner (spring 2026) but will still cap at ~128 GB.

6. **The refurbished market has nothing relevant** for this use case as of February 2026.

7. **Image recognition works fine** on Apple Silicon. Image *generation* is 2-8x slower than NVIDIA, but per your requirements, image generation is not needed (cloud fallback acceptable).
