# AI Inference Hardware — Benchmark Research

> Research compiled February 2026. All figures are approximate and depend heavily on context length, batch size, quantization method, driver version, and inference software. Numbers represent typical real-world results, not cherry-picked peaks.

---

## Table of Contents

1. [Token Generation Speed by Hardware](#1-token-generation-speed-by-hardware)
2. [Concurrent Inference Performance](#2-concurrent-inference-performance)
3. [Memory Requirements for Specific Models](#3-memory-requirements-for-specific-models)
4. [Inference Software Comparison](#4-inference-software-comparison)
5. [Minimum Hardware for Target Performance](#5-minimum-hardware-for-target-performance)
6. [Sources](#sources)

---

## 1. Token Generation Speed by Hardware

### 1.1 70B-Class Models (Llama 3.1 70B, Qwen2.5-Coder 72B, etc.)

All figures are single-stream token generation (tok/s) unless noted. Context ~4K tokens.

| Hardware | Q4_K_M (~4-bit) | Q8 (~8-bit) | FP16 | Notes |
|---|---|---|---|---|
| **RTX 5090** (32GB) | ~25-30 (partial offload) | Does not fit | Does not fit | 70B Q4 needs ~42GB; does not fit in 32GB without aggressive 2-3 bit quant or CPU offload. With IQ3 quant: ~20-25 tok/s |
| **2x RTX 5090** (64GB) | **~27 tok/s** | ~15-18 (estimated) | Does not fit | Ollama benchmarks: up to 27 tok/s on 70B. Matches H100 single-GPU perf. Sweet spot for 70B hosting |
| **RTX PRO 6000 Blackwell** (96GB) | ~29-32 | ~18-22 | ~10-14 | 96GB GDDR7; fits 70B Q4 entirely. 1,792 GB/s bandwidth. Dense 70B limited by reading all params. |
| **RTX 4090** (24GB) | ~15-20 (heavy offload) | Does not fit | Does not fit | 70B Q4 needs ~42GB; heavy CPU offload required. With IQ2 quant: ~50+ tok/s reported but severe quality loss |
| **2x RTX 4090** (48GB) | ~25-30 | Does not fit | Does not fit | Layer-split via llama.cpp; sequential processing, not true tensor parallel |
| **4x RTX 4090** (96GB) | ~40-50 | ~25-30 | Does not fit | vLLM with TP=4 + FP8: ~250 tok/s total throughput (batched) |
| **RTX 3090** (24GB) | ~12-15 (heavy offload) | Does not fit | Does not fit | ~16% slower than 4090 at same task. Similar offload constraints |
| **2x RTX 3090** (48GB) | ~20-25 | Does not fit | Does not fit | NVLink improves throughput 40-60% vs PCIe-only |
| **RTX A6000** (48GB) | ~30-35 | Does not fit (with KV cache) | Does not fit | 48GB can hold 70B Q4 (~42GB) with minimal KV cache headroom |
| **2x RTX A6000** (96GB) | ~40-50 | ~25-30 | Does not fit | Comfortable Q4 fit; tight for Q8 with KV cache |
| **4x RTX A6000** (192GB) | ~60-70 (estimated) | ~40-50 | ~25-30 | vLLM: 420-470 tok/s total throughput (batched, multi-user) |
| **A100 80GB** (single) | ~35-40 | ~25-30 | Does not fit | 80GB fits Q4 + generous KV cache. Q8 tight with context |
| **2x A100 80GB** (160GB) | ~50-60 | ~35-45 | ~20-25 | Strong production-grade option |
| **Mac Studio M2 Ultra** (192GB) | **~12-14 tok/s** | ~8-10 | ~4.7 tok/s | Memory bandwidth ~800 GB/s is the bottleneck. llama.cpp Metal backend |
| **Mac Studio M3 Ultra** (192GB) | **~15-20 tok/s** | ~10-14 | ~6-8 | ~800 GB/s bandwidth. MLX achieves ~30-50% faster than llama.cpp |
| **Mac Studio M3 Ultra** (512GB) | ~15-20 | ~10-14 | ~6-8 | Same speed as 192GB (bandwidth-limited), but can fit larger models |

**Key Insight — Dense 70B models:** For a single-stream 60-80 tok/s target on a dense 70B model at Q4, consumer/prosumer hardware tops out at ~32 tok/s (RTX PRO 6000 or 2x RTX 5090). However, see Section 1.3 — **MoE coding models are the practical path to 60-80+ tok/s** on this hardware.

### 1.2 7B-32B Models (Sub-Agent Workloads)

Single-stream token generation (tok/s), context ~4K tokens.

| Hardware | 7B Q4 | 7B Q8/FP16 | 14B Q4 | 32B Q4 | Notes |
|---|---|---|---|---|---|
| **RTX 5090** (32GB) | ~200+ | ~130-150 | ~120-140 | **~61** | 32GB fits all these models entirely in VRAM |
| **RTX PRO 6000 Blackwell** (96GB) | ~200+ | ~130-160 | ~120-150 | **~56-64** | 96GB fits all these models plus a 70B model simultaneously |
| **RTX 4090** (24GB) | ~130-150 | ~50-55 (FP16) | ~80-90 | ~35-45 | 24GB fits up to 32B Q4 with modest context |
| **RTX 3090** (24GB) | ~110-120 | ~45-50 (FP16) | ~65-75 | ~30-38 | Similar to 4090, ~16% slower |
| **RTX A6000** (48GB) | ~100-120 | ~50-60 | ~65-80 | ~35-45 | Lower clock speeds vs consumer GPUs; more VRAM headroom |
| **A100 80GB** | ~140-165 | ~80-100 | ~90-110 | ~50-60 | Strong all-around; HBM2e bandwidth advantage |
| **Mac M2 Ultra** (192GB) | ~115 (llama.cpp) / ~230 (MLX) | ~80-100 | ~60-80 | ~30-40 | MLX significantly faster than llama.cpp on Apple Silicon |
| **Mac M3 Ultra** (192GB) | ~120-130 (llama.cpp) / ~240+ (MLX) | ~90-110 | ~70-90 | ~35-50 | Similar bandwidth to M2 Ultra; slight arch improvements |

**Key Insight:** For 5x concurrent streams at 15-20 tok/s on 7B-32B models, a single RTX 4090/5090 can handle this comfortably for 7B models (130+ tok/s / 5 streams = 26+ tok/s each). For 32B models, you need at least 2 GPUs or accept lower per-stream rates.

### 1.3 MoE Coding Models — The Path to 60-80+ tok/s

Since early 2025, frontier coding models use **Mixture of Experts (MoE)** architecture — only a fraction of parameters are activated per token. This dramatically changes the performance picture:

**RTX PRO 6000 Blackwell (96 GB, single card, Ollama):**

| Model | Type | Total/Active Params | Quant | tok/s |
|---|---|---|---|---|
| **GPT-OSS-120B** | MoE | 120B / ~20B | Q4 | **134** |
| **GPT-OSS-20B** | MoE | 20B / ~5B | Q4 | **185** |
| **Qwen3-30B-A3B** | MoE | 30B / 3.3B | Q4 | **~150-200** (est.) |
| DeepSeek-R1 | Dense | 32B / 32B | Q4 | 64 |
| LLaMA 3.3 | Dense | 70B / 70B | Q4 | 32 |
| Qwen 2.5 | Dense | 72B / 72B | Q4 | 29 |

**Mac Studio M3 Ultra (256 GB, Olares benchmark suite):**

| Model | Type | Total/Active Params | 1 concurrent | 8 concurrent |
|---|---|---|---|---|
| **GPT-OSS-120B** | MoE | 120B / ~20B | **69.4 tok/s** | 19.1 tok/s |
| **Qwen3-30B-A3B** | MoE | 30B / 3.3B | **84 tok/s** | 24.9 tok/s |

**Key Insight:** MoE models make the 60-80 tok/s target easily achievable. A single RTX PRO 6000 runs GPT-OSS-120B at 134 tok/s — far exceeding the target. Even the Mac Studio M3 Ultra reaches ~69 tok/s with MoE models. Dense 70B models at ~29-32 tok/s are now the fallback, not the primary workload.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [Olares Blog Benchmarks](https://blog.olares.com/local-ai-hardware-performance-benchmarking/)

---

## 2. Concurrent Inference Performance

### 2.1 How tok/s Degrades with Multiple Concurrent Requests

General rule of thumb:
- **1 to max_batch_size requests:** Total system throughput increases roughly linearly; per-request tok/s decreases proportionally
- **Beyond max_batch_size:** Requests queue; latency spikes unpredictably
- **GPU saturation point:** Once CUDA cores and memory bandwidth are fully utilized, adding more concurrent requests only increases latency

**Measured degradation patterns:**
- vLLM on A100: 1 request = ~40 tok/s per request; 10 concurrent = ~15 tok/s per request; 50 concurrent = ~8 tok/s per request (70B Q4)
- SGLang: More consistent under load; maintains ~30-31 tok/s per request across concurrent 70B-FP8 requests while vLLM drops from 22 to 16 tok/s
- llama.cpp server: Performance degrades in steps aligned with slot count; exceeding slots causes unpredictable latency spikes

### 2.2 vLLM Batched Inference Throughput (Multi-GPU)

| Setup | Model | Total Throughput | Concurrent Users | Per-User tok/s |
|---|---|---|---|---|
| 4x A6000 (vLLM, TP=4) | Qwen2.5 72B | ~450 tok/s | 10-50 | ~9-45 |
| 4x RTX 4090 (vLLM, TP=4, FP8) | Llama 70B | ~250 tok/s | 10-30 | ~8-25 |
| 2x A100 80GB (vLLM, TP=2) | Llama 70B | ~200-300 tok/s | 10-30 | ~7-30 |
| 1x A100 80GB (vLLM) | Qwen 14B | ~3,000 tok/s | 50-100 | ~30-60 |
| 1x RTX 4090 (vLLM) | Llama 8B (FP16) | ~340 tok/s | 5-10 | ~34-68 |
| 1x RTX 4090 (vLLM) | Llama 8B (GPTQ 4-bit) | ~600 tok/s | 5-15 | ~40-120 |

### 2.3 llama.cpp Server Mode with Parallel Slots

Benchmark from December 2025 study (A40 GPU, coding workload):

| Parallel Slots | Sustainable Concurrent Users | Latency Behavior |
|---|---|---|
| 1 | 1-3 | Low, predictable |
| 4 | 4-8 | Moderate, stable |
| 8 | 8-15 | Acceptable, some variation |
| 16+ | 15-22 | Escalating, unpredictable |

**Key findings:**
- ~22 concurrently active coding developers per A40 GPU (48GB, lower performance than A100)
- Optimal: no more than 8 concurrent requests per GPU for acceptable latency
- Once requests exceed slots, competition for shared resources causes stepwise latency growth

### 2.4 Running Multiple Models Simultaneously on Different GPUs

**Yes, this is fully supported.** Strategies:

1. **Separate Ollama instances:** Run two `ollama serve` processes, each pinned to a specific GPU via `CUDA_VISIBLE_DEVICES`. E.g., GPU 0 runs 70B model, GPU 1 runs 7B model. Use different ports. Put a reverse proxy (Nginx, LiteLLM) in front.

2. **vLLM with Ray Serve:** Deploy multiple vLLM workers, each assigned to specific GPUs. One worker serves the 70B model (TP across GPUs 0-1), another serves the 7B model on GPU 2.

3. **llama.cpp:** Run multiple `llama-server` instances on different ports, each with `--gpu-layers` targeting different GPUs.

4. **Ollama's built-in:** `OLLAMA_MAX_LOADED_MODELS` controls concurrent model count (default: 3x number of GPUs). Ollama can auto-distribute models across GPUs.

**Example multi-model config (4x GPU system):**
- GPUs 0-1: 70B Q4 model (tensor parallel, ~42GB used)
- GPU 2: 32B Q4 sub-agent model (~18GB used)
- GPU 3: 7B vision model (~4-8GB used) + 7B sub-agent model (~4GB used)

---

## 3. Memory Requirements for Specific Models

### 3.1 Qwen2.5-Coder 72B

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | Total Estimate |
|---|---|---|---|---|
| **Q4_K_M** | ~36 GB | ~2.5 GB | ~10 GB | **~39-46 GB** |
| **Q8** | ~72 GB | ~2.5 GB | ~10 GB | **~75-82 GB** |
| **FP16** | ~144 GB | ~2.5 GB | ~10 GB | **~147-154 GB** |

### 3.2 DeepSeek-V3 / DeepSeek-R1

These are 671B MoE models (37B active parameters per token). They are NOT feasible for local consumer hardware.

| Quantization | Model Weights | Minimum VRAM | Recommended Setup |
|---|---|---|---|
| **FP16** | ~1,343 GB | ~1,543 GB | 16x+ H100 80GB |
| **FP8** | ~685 GB | ~750 GB | 8x H100 80GB |
| **Q4 (AWQ)** | ~386 GB | ~420 GB | 8x A100 80GB or 5x H100 |
| **1.58-bit (Unsloth)** | ~131 GB | ~160 GB | 2x H100 80GB |

**Verdict:** DeepSeek-V3/R1 full models are out of scope for the budget/hardware class discussed. The distilled versions (e.g., DeepSeek-R1-Distill-Qwen-32B at ~18GB Q4) are the practical option.

### 3.3 DeepSeek-Coder-V2

| Variant | Parameters (Total / Active) | FP16 VRAM | Q4 Estimate | Q8 Estimate |
|---|---|---|---|---|
| **Lite** | 16B / 2.4B active | ~32 GB | ~8-10 GB | ~16-18 GB |
| **Full** | 236B / 21B active | ~480 GB (8x 80GB GPUs) | ~120 GB | ~240 GB |

**Verdict:** DeepSeek-Coder-V2 Lite (16B) is practical on consumer hardware. The full 236B model requires datacenter-class hardware. For coding, Qwen2.5-Coder 72B is likely a better choice for local deployment.

### 3.4 Llama 3.1 70B

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | + KV Cache (128K ctx) | Total Estimate |
|---|---|---|---|---|---|
| **Q4_K_M** | ~34 GB | ~2.5 GB | ~10 GB | ~40 GB | **~37-74 GB** |
| **Q8** | ~69 GB | ~2.5 GB | ~10 GB | ~40 GB | **~72-109 GB** |
| **FP16** | ~138 GB | ~2.5 GB | ~10 GB | ~40 GB | **~141-178 GB** |

**Important:** Llama 3.1's 128K native context window can consume enormous KV cache memory. At 128K context, KV cache alone is ~40GB in FP16. Use GQA (Grouped Query Attention) and KV cache quantization to reduce this.

### 3.5 Vision Models for Screenshot QA

| Model | Parameters | VRAM (FP16) | VRAM (Q4/INT4) | DocVQA Score | Best For |
|---|---|---|---|---|---|
| **Qwen2.5-VL-7B** | 7B | ~13 GB | ~4-5 GB | High | Best overall quality/size ratio for vision |
| **Qwen3-VL-8B** | 8B | ~16 GB | ~5-6 GB | Very High | Latest generation, best quality at this size |
| **Llama 3.2 Vision 11B** | 11B | ~22 GB | ~7-8 GB | 88.4% | Good all-around; strong document understanding |
| **Qwen2.5-VL-3B** | 3B | ~6 GB | ~2-3 GB | Good | Ultra-lightweight; fits alongside larger models |
| **Qwen3-VL-2B** | 2B | ~4 GB | ~1.5-2 GB | Decent | Minimal footprint |
| **PaddleOCR-VL-0.9B** | 0.9B | ~2 GB | <1 GB | Good (OCR-specific) | Pure OCR extraction |
| **Granite Vision 3.3 2B** | 2B | ~4 GB | ~1.5-2 GB | Decent | IBM's compact vision model |

**Recommendation for your use case:** Qwen2.5-VL-7B or Qwen3-VL-8B provides excellent screenshot QA quality in ~5-6GB (Q4), easily fitting alongside a 70B coding model. For minimal footprint, Qwen2.5-VL-3B at ~3GB Q4 is surprisingly capable.

---

## 4. Inference Software Comparison

### 4.1 Feature Matrix

| Feature | llama.cpp | vLLM | SGLang | Ollama | MLX |
|---|---|---|---|---|---|
| **Multi-GPU Tensor Parallel** | No (layer split only) | Yes (TP + PP) | Yes (TP + DP) | No (layer split) | No (single SoC) |
| **Concurrent Requests** | Yes (parallel slots) | Yes (continuous batching) | Yes (continuous batching) | Yes (limited) | Yes (vllm-mlx) |
| **Throughput at Scale** | Low-Medium | **Very High** | **Very High** | Low | Medium |
| **Single-User Latency** | **Very Low** | Low | Low | Low | **Very Low** |
| **Apple Silicon** | Yes (Metal) | Partial (vllm-mlx) | No | Yes (Metal) | **Yes (native)** |
| **GGUF Quantization** | **Yes (native)** | No (AWQ/GPTQ) | No (AWQ/GPTQ) | Yes (via llama.cpp) | Yes (MLX format) |
| **Vision Models** | Yes | Yes | Yes | Yes | Yes |
| **Ease of Setup** | Medium | Medium | Medium | **Very Easy** | Easy |
| **OpenAI-Compatible API** | Yes | Yes | Yes | Yes | Yes (vllm-mlx) |
| **Memory Efficiency** | Good | **Very Good (PagedAttention)** | **Very Good (RadixAttention)** | Good | Good |

### 4.2 When to Use What

**For multi-user / multi-agent serving (your use case):**
1. **vLLM** or **SGLang** — Best choices. vLLM has the largest ecosystem; SGLang often matches or exceeds vLLM throughput and handles concurrent requests more consistently (30-31 tok/s stable vs vLLM's 22 -> 16 tok/s degradation under load).
2. **Ollama** — Good for development and simple setups. Easy multi-model with `OLLAMA_MAX_LOADED_MODELS`. Becomes a bottleneck at high concurrency.
3. **llama.cpp** — Best for single-user, maximum-latency-optimized workloads. Server mode works but scales poorly vs vLLM/SGLang.

**Multi-GPU tensor parallelism:**
- **vLLM:** Full tensor parallelism + pipeline parallelism. Set `tensor_parallel_size=N`. Requires NVIDIA GPUs.
- **SGLang:** Tensor parallelism + data parallelism. Strong multi-GPU scaling.
- **llama.cpp:** Only layer splitting (sequential, not parallel). Each GPU processes layers in sequence. Works, but 30-50% slower than true tensor parallel.
- **Ollama:** Uses llama.cpp backend; same layer-split limitation.

**Apple Silicon (MLX vs llama.cpp Metal):**
- **MLX is 30-50% faster** than llama.cpp Metal on Apple Silicon. MLX's zero-copy unified memory operations and lazy evaluation give it a significant edge.
- **vllm-mlx** (community project): Brings continuous batching to Apple Silicon. 21-87% higher throughput than llama.cpp. Supports OpenAI/Anthropic-compatible API. Supports vision models.
- **Recommendation:** Use MLX-based tools (mlx-lm, vllm-mlx, or LM Studio with MLX backend) on Apple Silicon. Do not use llama.cpp Metal unless you need a specific GGUF model not available in MLX format.

### 4.3 Performance Comparison: Throughput at Scale

Measured on similar hardware (A100-class), Llama 70B, high concurrency:

| Engine | Relative Throughput | Notes |
|---|---|---|
| vLLM | 1.0x (baseline) | PagedAttention, continuous batching |
| SGLang | 1.1-1.3x | RadixAttention; excels with shared prefixes (up to 158K tok/s with 75% cache hit) |
| TensorRT-LLM | 1.5-1.7x | Requires NVIDIA GPUs, complex setup, model-specific compilation |
| llama.cpp | 0.03-0.1x | Not optimized for high-concurrency GPU serving |
| Ollama | 0.05-0.1x | Built on llama.cpp; 41 TPS peak vs vLLM's 793 TPS |

---

## 5. Minimum Hardware for Target Performance

### 5.1 Target Recap (from REQUIREMENTS.md)

| Requirement | Target | Minimum |
|---|---|---|
| Main agent speed | 60-80 tok/s | 30 tok/s |
| Sub-agent speed (7B-32B) | 15-20 tok/s each | ~10 tok/s |
| Concurrent streams | 5-10 | 3 |
| Image recognition | Concurrent with LLM | -- |

**MoE update:** The 60-80 tok/s target refers primarily to MoE coding models (e.g., GPT-OSS-120B), not dense 70B models. See Section 1.3 for MoE benchmarks. Dense 70B models serve as fallbacks for specific tasks.

### 5.2 Reality Check: 60-80 tok/s — Achievable with MoE Models

**Update (February 2026):** The original analysis below was written for dense 70B models. With the industry shift to MoE coding models, the 60-80 tok/s target is now achievable on prosumer hardware:

- **RTX PRO 6000 Blackwell** (single card): **134 tok/s** on GPT-OSS-120B (MoE) — far exceeds target
- **Mac Studio M3 Ultra**: **~69 tok/s** on GPT-OSS-120B (MoE) — meets target
- Dense 70B models remain at ~29-32 tok/s on the best single-GPU hardware

**For dense 70B models only** (the original analysis remains valid):
- Single A100 80GB: ~35-40 tok/s on 70B Q4 (single stream)
- Single H100 80GB: ~60-80 tok/s on 70B Q4 — the only single-GPU option
- 2x RTX 5090: ~27 tok/s (Ollama)
- RTX PRO 6000: ~29-32 tok/s
- Mac M3 Ultra: ~15-20 tok/s (MLX, Q4)

**Bottom line:** Use MoE coding models (GPT-OSS-120B, Qwen3-Coder-30B-A3B) as primary workloads. Dense 70B models serve as fallbacks for specific tasks where MoE models underperform.

### 5.3 Configuration Options for Your Requirements

#### Option NEW: Single RTX PRO 6000 Blackwell (RECOMMENDED)

| Aspect | Details |
|---|---|
| **GPU** | 1x NVIDIA RTX PRO 6000 Blackwell (96 GB GDDR7, 1,792 GB/s) |
| **MoE main agent** | GPT-OSS-120B Q4 (~65 GB): **~134 tok/s** — far exceeds 60-80 target |
| **MoE sub-agents** | Qwen3-Coder-30B-A3B Q4 (~18 GB): **~150-200 tok/s** |
| **Dense 70B fallback** | Qwen2.5-Coder 72B Q4 (~36 GB): ~29-32 tok/s |
| **Vision** | Qwen2.5-VL-7B Q4 (~5 GB) concurrent with above |
| **Concurrent** | All models loaded simultaneously on single card (~88 GB, 8 GB free) |
| **Cost** | Single PRO 6000: CHF 6,443. Full system: ~CHF 8,200 |
| **Noise** | Quiet — single blower card, 300W TDP |
| **Power** | ~400W under full load |
| **Verdict** | **Best overall option.** MoE models make a single 96 GB card sufficient for all targets. |

#### Option A: 4x RTX A6000 (48GB each = 192GB total VRAM)

| Aspect | Details |
|---|---|
| **GPU allocation** | GPUs 0-2: 70B Q4 (TP=3 via vLLM) / GPU 3: 32B Q4 sub-agent + 7B vision model |
| **70B main agent** | ~40-50 tok/s single stream (vLLM, TP=3) |
| **Sub-agents** | 5x concurrent 7B Q4: ~20+ tok/s each on GPU 3 |
| **Vision** | Qwen2.5-VL-7B Q4 on GPU 3 alongside sub-agent model (~10GB total) |
| **Concurrent** | Yes — separate vLLM instances per GPU group |
| **Cost** | 4x A6000 used: ~$3,000-5,000. Full system: ~$6,000-9,000 |
| **Noise** | Blower-style coolers; LOUD. Need aftermarket cooling or case with good airflow |
| **Power** | ~1,200-1,400W under load |

#### Option B: 2x RTX 5090 (32GB each = 64GB total VRAM)

| Aspect | Details |
|---|---|
| **GPU allocation** | Both GPUs: 70B Q4 (layer split, ~42GB) / Remaining ~22GB: 7B models |
| **70B main agent** | ~25-27 tok/s single stream |
| **Sub-agents** | Need CPU offload or smaller models; limited concurrent capacity |
| **Vision** | Tight — ~5GB headroom after 70B model |
| **Cost** | 2x RTX 5090: ~$4,000. Full system: ~$5,500-7,000 |
| **Verdict** | **Insufficient for your requirements.** 64GB is too tight for 70B + sub-agents + vision concurrently |

#### Option C: Mac Studio M3 Ultra (192GB unified memory, $5,599-$7,999)

| Aspect | Details |
|---|---|
| **GPU allocation** | All models share 192GB unified memory pool |
| **70B main agent** | ~15-20 tok/s Q4 (MLX). ~10-14 tok/s Q8 |
| **Sub-agents** | 7B models: ~40-60 tok/s each; 5 concurrent: ~8-12 tok/s each |
| **Vision** | Qwen2.5-VL-7B easily fits in remaining memory |
| **Concurrent** | Limited by ~800 GB/s memory bandwidth shared across all models |
| **Cost** | 192GB: ~$5,599. 512GB: ~$8,999 (buy from Apple Store CH) |
| **Noise** | **Near silent** |
| **Power** | ~200-300W |
| **Verdict** | Hits minimum (30 tok/s) only if 70B speed is acceptable at ~15-20 tok/s. **Falls short of 30 tok/s minimum for 70B target.** Excellent for sub-agents and concurrent model hosting. Best noise/power profile |

#### Option D: 4x RTX 4090 (24GB each = 96GB total VRAM)

| Aspect | Details |
|---|---|
| **GPU allocation** | GPUs 0-3: 70B Q4 (TP=4 via vLLM, needs FP8 or Q4 AWQ) / OR GPUs 0-1: 70B, GPUs 2-3: sub-agents |
| **70B main agent** | TP=4: ~40-50 tok/s. TP=2: ~25-30 tok/s (frees GPUs 2-3) |
| **Sub-agents** | If TP=2 for 70B: GPUs 2-3 each run 7B-32B models. 5x 7B concurrent: ~25+ tok/s each |
| **Vision** | Fits on GPU 2 or 3 alongside sub-agent models |
| **Cost** | 4x RTX 4090: ~$6,400-8,000. Full system: ~$8,500-11,000 |
| **Noise** | High — 4 triple-fan GPUs at load |
| **Power** | ~1,600-1,800W peak. Needs dedicated circuit |
| **SLI/NVLink** | RTX 4090 does NOT support NVLink. Multi-GPU is PCIe only |
| **Verdict** | Strong option if you can manage power and noise. Hits ~40-50 tok/s on 70B with all 4 GPUs, or ~25-30 tok/s with 2 GPUs while freeing 2 GPUs for sub-agents |

#### Option E: 2x RTX A6000 (48GB each = 96GB) + 1x RTX 4090 (24GB)

| Aspect | Details |
|---|---|
| **GPU allocation** | A6000 pair: 70B Q4 (vLLM TP=2) / RTX 4090: sub-agents + vision |
| **70B main agent** | ~35-45 tok/s (vLLM TP=2) |
| **Sub-agents** | RTX 4090: 5x 7B concurrent at ~25+ tok/s each; or 2x 32B at ~15-18 tok/s each |
| **Vision** | Qwen2.5-VL-7B Q4 on RTX 4090 (~5GB) alongside sub-agents |
| **Cost** | 2x A6000 used: ~$2,000-3,000. RTX 4090: ~$1,600-2,000. System: ~$5,500-8,000 |
| **Verdict** | Good balance of performance and cost. Meets minimum targets |

#### Option F: 2x A100 80GB (160GB total VRAM) — Refurbished/Used

| Aspect | Details |
|---|---|
| **GPU allocation** | GPU 0: 70B Q4 (~42GB) + sub-agents / GPU 1: 70B Q4 overflow + sub-agents + vision. OR TP=2 for 70B across both |
| **70B main agent** | TP=2: ~50-60 tok/s. Single GPU: ~35-40 tok/s |
| **Sub-agents** | Remaining VRAM (~38GB per GPU after 70B in TP=2): multiple 7B/14B models |
| **Vision** | Easily fits |
| **Cost** | 2x A100 80GB used: ~$5,000-8,000. System: ~$7,000-12,000 |
| **Noise** | Designed for datacenter — **extremely loud** without modification |
| **Power** | ~600-800W for GPUs alone |
| **Verdict** | Closest to hitting the 60 tok/s target. But noise is a serious problem for home office. Need aftermarket cooling solution or accept server noise |

### 5.4 Summary Comparison

| Config | 70B tok/s (single) | Sub-agent capacity | Vision concurrent | Total VRAM | Est. System Cost | Noise | Power |
|---|---|---|---|---|---|---|---|
| RTX PRO 6000 (MoE) | ~134 (MoE) / ~32 (dense) | Excellent | Yes | 96 GB | ~$8.2K CHF | Quiet | ~400W |
| 4x A6000 | ~40-50 | Excellent | Yes | 192 GB | $6K-9K | Loud | ~1,400W |
| 4x RTX 4090 | ~40-50 (TP=4) | Good (if TP=2) | Yes | 96 GB | $8.5K-11K | Loud | ~1,800W |
| 2x A100 80GB | ~50-60 | Excellent | Yes | 160 GB | $7K-12K | Very Loud | ~800W |
| 2x A6000 + 4090 | ~35-45 | Good | Yes | 120 GB | $5.5K-8K | Moderate | ~1,000W |
| Mac M3 Ultra 192GB | ~15-20 | Good | Yes | 192 GB | $5.6K-8K | Silent | ~300W |
| 2x RTX 5090 | ~25-27 | Limited | Tight | 64 GB | $5.5K-7K | Moderate | ~700W |

### 5.5 Recommendation

**For your specific requirements (home office, multi-agent coding, concurrent models, 4K-15K CHF budget):**

**Best overall: Single RTX PRO 6000 Blackwell (~CHF 8,200 full system)**
- 96 GB VRAM on a single card
- MoE main agent (GPT-OSS-120B) at ~134 tok/s — far exceeds 60-80 target
- MoE sub-agents at ~150-200 tok/s
- Dense 70B fallback at ~29-32 tok/s
- Quiet, low power (~400W), fits standard outlets
- Upgrade path: add 2nd PRO 6000 later for 192 GB

**Best if noise/power matter: Mac Studio M3 Ultra 256GB (~CHF 6,100)**
- Silent, energy-efficient, turnkey
- MoE main agent at ~69 tok/s — meets target
- 256 GB unified memory handles all models concurrently
- Tradeoff: ~half the MoE speed of PRO 6000, concurrent performance degrades faster

**Best for zero bandwidth contention: RTX PRO 6000 + RTX 5090 (~CHF 11,450)**
- 128 GB total VRAM across 2 dedicated GPUs
- Main agent and sub-agents on separate GPUs — no sharing

**Note:** The previous recommendations (4x RTX A6000, 2x A100, etc.) were based on dense 70B models only. With MoE coding models, the single RTX PRO 6000 is the clear winner — delivering 4x the speed of dense 70B at lower cost, noise, and power.

---

## Sources

### GPU Benchmarks & Rankings
- [Hardware Corner — Definitive GPU Ranking for LLMs](https://www.hardware-corner.net/gpu-ranking-local-llm/)
- [Hardware Corner — RTX 5090 LLM Benchmarks](https://www.hardware-corner.net/rtx-5090-llm-benchmarks/)
- [CloudRift — RTX 4090 vs 5090 vs PRO 6000 Benchmark](https://www.cloudrift.ai/blog/benchmarking-rtx-gpus-for-llm-inference)
- [CloudRift — Benchmarking GPU Servers for LLM Inference](https://www.cloudrift.ai/blog/benchmarking-gpu-servers-for-llm-inference)
- [DatabaseMart — 2x RTX 5090 Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2)
- [DatabaseMart — RTX 5090 Ollama Benchmark (single)](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090)
- [DatabaseMart — 4x A6000 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-a6000-4)
- [DatabaseMart — RTX 4090 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-rtx4090)
- [DatabaseMart — Dual A100 Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-dual-a100)
- [RunPod — RTX 5090 LLM Benchmarks](https://www.runpod.io/blog/rtx-5090-llm-benchmarks)
- [XiongjieDai — GPU Benchmarks on LLM Inference (GitHub)](https://github.com/XiongjieDai/GPU-Benchmarks-on-LLM-Inference)
- [LocalLLM.in — Best GPUs for LLM Inference 2025](https://localllm.in/blog/best-gpus-llm-inference-2025)

### Apple Silicon
- [Apple — Mac Studio M3 Ultra Announcement](https://www.apple.com/newsroom/2025/03/apple-unveils-new-mac-studio-the-most-powerful-mac-ever/)
- [llama.cpp Discussion #4167 — Apple Silicon Performance](https://github.com/ggml-org/llama.cpp/discussions/4167)
- [Kunar — Benchmarking MLX vs llama.cpp (Medium)](https://medium.com/@andreask_75652/benchmarking-apples-mlx-vs-llama-cpp-bbbebdc18416)
- [arXiv:2511.05502 — Comparative Study: MLX, Ollama, llama.cpp on Apple Silicon](https://arxiv.org/abs/2511.05502)
- [Markus Schall — MLX on Apple Silicon vs Ollama](https://www.markus-schall.de/en/2025/11/apple-mlx-vs-nvidia-how-local-ki-inference-works-on-the-mac/)
- [Scalastic — Apple Silicon vs NVIDIA CUDA AI Comparison 2025](https://scalastic.io/en/apple-silicon-vs-nvidia-cuda-ai-2025/)
- [Billy Newport — M3 Ultra Mac Studio LLM Inference (Medium)](https://medium.com/@billynewport/apples-m3-ultra-mac-studio-misses-the-mark-for-llm-inference-f57f1f10a56f)
- [Xenix Blog — Mac Studio 2025 vs NVIDIA Blackwell](https://xenix.blog/2025/05/05/mac-studio-m4-max-m3-ultra-vs-nvidia-blackwell-which-desktop-reigns-for-local-genai/)
- [arXiv:2601.19139 — Native LLM Inference at Scale on Apple Silicon](https://arxiv.org/html/2601.19139)

### Inference Engines
- [Red Hat — vLLM or llama.cpp: Choosing the Right Engine](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case)
- [Red Hat — Ollama vs vLLM Performance Benchmarking](https://developers.redhat.com/articles/2025/08/08/ollama-vs-vllm-deep-dive-performance-benchmarking)
- [vLLM Blog — v0.6.0 Performance Update](https://blog.vllm.ai/2024/09/05/perf-update.html)
- [vLLM Blog — Blackwell InferenceMAX](https://blog.vllm.ai/2025/10/09/blackwell-inferencemax.html)
- [vLLM Docs — Parallelism and Scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)
- [AI Multiple Research — vLLM vs LMDeploy vs SGLang](https://research.aimultiple.com/inference-engines/)
- [GPU-Mart — SGLang vs vLLM Comparison](https://www.gpu-mart.com/blog/sglang-vs-vllm)
- [BentoML — Benchmarking LLM Inference Backends](https://www.bentoml.com/blog/benchmarking-llm-inference-backends)
- [ITECS — vLLM vs Ollama vs llama.cpp vs TGI 2025 Guide](https://itecsonline.com/post/vllm-vs-ollama-vs-llama.cpp-vs-tgi-vs-tensort)
- [Clarifai — SGLang vs vLLM vs TensorRT-LLM](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b)
- [waybarrios — vllm-mlx for Apple Silicon (GitHub)](https://github.com/waybarrios/vllm-mlx)
- [Ferrari — llama.cpp Parallelism on A40 GPUs (Medium)](https://medium.com/@ferraricorneloup.teo/how-many-developers-can-one-gpu-serve-benchmarking-llama-cpp-parallelism-on-a40-gpus-0ea2a8c36045)

### Model VRAM Requirements
- [APXML — GPU Requirements for Qwen Models](https://apxml.com/posts/gpu-system-requirements-qwen-models)
- [APXML — GPU Requirements for Llama 3 Models](https://apxml.com/posts/ultimate-system-requirements-llama-3-models)
- [APXML — GPU Requirements for DeepSeek Models](https://apxml.com/posts/system-requirements-deepseek-models)
- [LocalLLM.in — Ollama VRAM Requirements Guide 2026](https://localllm.in/blog/ollama-vram-requirements-for-local-llms)
- [LocalAI.computer — Llama 3.1 70B VRAM Requirements](https://www.localai.computer/models/meta-llama-llama-3-1-70b-instruct)
- [Novita AI — Qwen2.5-VL-72B VRAM Needs](https://blogs.novita.ai/qwen2-5-vl-72b-vram/)
- [DatabaseMart — How Much VRAM for 7B-70B LLMs](https://www.databasemart.com/blog/how-much-vram-do-you-need-for-7-70b-llm)
- [Unsloth — DeepSeek-R1 Dynamic 1.58-bit](https://unsloth.ai/blog/deepseekr1-dynamic)

### Vision Models
- [Hugging Face — Qwen2.5-VL-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- [QwenLM — Qwen3-VL (GitHub)](https://github.com/QwenLM/Qwen3-VL)
- [Meta AI — Llama 3.2 Vision](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/)
- [Clarifai — Benchmarking Open-Source Vision LLMs](https://www.clarifai.com/blog/benchmarking-best-open-source-vision-language-models)
- [Modal — 8 Top Open-Source OCR Models Compared](https://modal.com/blog/8-top-open-source-ocr-models-compared)

### Concurrent Inference & Multi-Model
- [NVIDIA — LLM Inference Benchmarking Fundamental Concepts](https://developer.nvidia.com/blog/llm-benchmarking-fundamental-concepts/)
- [NVIDIA — Accelerating LLMs with llama.cpp on RTX Systems](https://developer.nvidia.com/blog/accelerating-llms-with-llama-cpp-on-nvidia-rtx-systems)
- [Ollama GitHub — Multi-GPU Issues](https://github.com/ollama/ollama/issues/7104)
- [arXiv:2511.17593 — Comparative Analysis of LLM Inference Serving](https://arxiv.org/html/2511.17593v1)
