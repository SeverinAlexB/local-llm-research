# AI Inference Hardware — Benchmarks & Technical Reference

> Research compiled February 2026. All figures are approximate and depend heavily on context length, batch size, quantization method, driver version, and inference software. Numbers represent typical real-world results, not cherry-picked peaks.

---

## Table of Contents

1. [Token Generation Speed by Hardware](#1-token-generation-speed-by-hardware)
2. [Concurrent Inference Performance](#2-concurrent-inference-performance)
3. [Memory Requirements for Specific Models](#3-memory-requirements-for-specific-models)
4. [Inference Software Comparison](#4-inference-software-comparison)
5. [Concurrent Inference Strategy](#5-concurrent-inference-strategy)
6. [Sources](#sources)

---

## 1. Token Generation Speed by Hardware

### 1.1 70B-Class Models (Llama 3.1 70B, Qwen2.5-Coder 72B, etc.)

All figures are single-stream token generation (tok/s) unless noted. Context ~4K tokens.

| Hardware | Q4_K_M (~4-bit) | Q8 (~8-bit) | FP16 | Notes |
|---|---|---|---|---|
| **RTX 5090** (32GB) | ~25-30 (partial offload) | Does not fit | Does not fit | 70B Q4 needs ~42GB; does not fit in 32GB without aggressive 2-3 bit quant or CPU offload |
| **2x RTX 5090** (64GB) | **~27 tok/s** | ~15-18 (estimated) | Does not fit | Ollama benchmarks: up to 27 tok/s on 70B |
| **RTX PRO 6000 Blackwell** (96GB) | ~29-32 | ~18-22 | ~10-14 | 96GB GDDR7; fits 70B Q4 entirely. 1,792 GB/s bandwidth |
| **RTX 4090** (24GB) | ~15-20 (heavy offload) | Does not fit | Does not fit | 70B Q4 needs ~42GB; heavy CPU offload required |
| **2x RTX 4090** (48GB) | ~25-30 | Does not fit | Does not fit | Layer-split via llama.cpp; sequential processing |
| **4x RTX 4090** (96GB) | ~40-50 | ~25-30 | Does not fit | vLLM with TP=4 + FP8: ~250 tok/s total throughput (batched) |
| **RTX A6000** (48GB) | ~30-35 | Does not fit (with KV cache) | Does not fit | 48GB can hold 70B Q4 (~42GB) with minimal KV cache headroom |
| **2x RTX A6000** (96GB) | ~40-50 | ~25-30 | Does not fit | Comfortable Q4 fit |
| **A100 80GB** (single) | ~35-40 | ~25-30 | Does not fit | 80GB fits Q4 + generous KV cache |
| **2x A100 80GB** (160GB) | ~50-60 | ~35-45 | ~20-25 | Strong production-grade option |
| **Mac Studio M3 Ultra** (192GB) | **~15-20 tok/s** | ~10-14 | ~6-8 | ~819 GB/s bandwidth. MLX achieves ~30-50% faster than llama.cpp |

**Key Insight — Dense 70B models:** For a single-stream 60-80 tok/s target on a dense 70B model at Q4, consumer/prosumer hardware tops out at ~32 tok/s (RTX PRO 6000 or 2x RTX 5090). See Section 1.3 — **MoE coding models are the practical path to 60-80+ tok/s**.

### 1.2 7B-32B Models (Sub-Agent Workloads)

Single-stream token generation (tok/s), context ~4K tokens.

| Hardware | 7B Q4 | 7B Q8/FP16 | 14B Q4 | 32B Q4 | Notes |
|---|---|---|---|---|---|
| **RTX 5090** (32GB) | ~200+ | ~130-150 | ~120-140 | **~61** | 32GB fits all these models entirely in VRAM |
| **RTX PRO 6000 Blackwell** (96GB) | ~200+ | ~130-160 | ~120-150 | **~56-64** | 96GB fits all these models plus a 70B model simultaneously |
| **RTX 4090** (24GB) | ~130-150 | ~50-55 (FP16) | ~80-90 | ~35-45 | 24GB fits up to 32B Q4 with modest context |
| **RTX 3090** (24GB) | ~110-120 | ~45-50 (FP16) | ~65-75 | ~30-38 | Similar to 4090, ~16% slower |
| **RTX A6000** (48GB) | ~100-120 | ~50-60 | ~65-80 | ~35-45 | Lower clock speeds vs consumer GPUs; more VRAM |
| **A100 80GB** | ~140-165 | ~80-100 | ~90-110 | ~50-60 | Strong all-around; HBM2e bandwidth advantage |
| **Mac M3 Ultra** (192GB) | ~120-130 (llama.cpp) / ~240+ (MLX) | ~90-110 | ~70-90 | ~35-50 | MLX significantly faster than llama.cpp |

### 1.3 MoE Coding Models — The Path to 60-80+ tok/s

Since early 2025, frontier coding models use **Mixture of Experts (MoE)** architecture — only a fraction of parameters are activated per token.

**RTX PRO 6000 Blackwell (96 GB, single card, Ollama):**

| Model | Type | Total/Active Params | Quant | tok/s |
|---|---|---|---|---|
| **GPT-OSS-120B** | MoE | 120B / ~20B | Q4 | **134** |
| **GPT-OSS-20B** | MoE | 20B / ~5B | Q4 | **185** |
| **Qwen3-30B-A3B** | MoE | 30B / 3.3B | Q4 | **~150-200** (est.) |
| DeepSeek-R1 | Dense | 32B / 32B | Q4 | 64 |
| Gemma 3 | Dense | 27B / 27B | Q4 | 62 |
| LLaMA 3.3 | Dense | 70B / 70B | Q4 | 32 |
| Qwen 2.5 | Dense | 72B / 72B | Q4 | 29 |

**Mac Studio M3 Ultra (256 GB, Olares benchmark suite):**

| Model | Type | Total/Active Params | 1 concurrent | 8 concurrent |
|---|---|---|---|---|
| **GPT-OSS-120B** | MoE | 120B / ~20B | **69.4 tok/s** | 19.1 tok/s |
| **Qwen3-30B-A3B** | MoE | 30B / 3.3B | **84 tok/s** | 24.9 tok/s |

**DGX Spark / ASUS Ascent (GB10, 128 GB per unit):**

| Model | Type | 1 unit tok/s | 2 units (TP=2) tok/s |
|---|---|---|---|
| **GPT-OSS-120B** | MoE | 50-55 | **75** |
| **GPT-OSS-20B** | MoE | 50 | — |
| **Llama 3.1 70B** | Dense (FP8) | 2.7 | ~5 |

**Key Insight:** MoE models make the 60-80 tok/s target easily achievable. Dense 70B models at ~29-32 tok/s are now the fallback, not the primary workload.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [Olares Blog Benchmarks](https://blog.olares.com/local-ai-hardware-performance-benchmarking/), [LMSYS DGX Spark Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)

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

### 2.4 Running Multiple Models Simultaneously on Different GPUs

**Yes, this is fully supported.** Strategies:

1. **Separate Ollama instances:** Run two `ollama serve` processes, each pinned to a specific GPU via `CUDA_VISIBLE_DEVICES`. Put a reverse proxy (Nginx, LiteLLM) in front.
2. **vLLM with Ray Serve:** Deploy multiple vLLM workers, each assigned to specific GPUs.
3. **llama.cpp:** Run multiple `llama-server` instances on different ports, each targeting different GPUs.
4. **Ollama's built-in:** `OLLAMA_MAX_LOADED_MODELS` controls concurrent model count (default: 3x number of GPUs).

---

## 3. Memory Requirements for Specific Models

### 3.1 MoE Models

| Model | Total / Active Params | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|---|
| GPT-OSS-120B | 120B / ~20B | Q4 | ~65 GB | ~2 GB | ~67 GB |
| Qwen3-Coder-30B-A3B | 30B / 3.3B | Q4 | ~18 GB | ~1 GB | ~19 GB |
| MiniMax-M2.5 | 230B / 10B | NVFP4 | ~130 GB | ~5 GB | ~135 GB |
| DeepSeek-Coder-V2-Lite | 16B / 2.4B | Q4 | ~10 GB | ~1 GB | ~11 GB |

### 3.2 Dense Models

| Model | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|
| Qwen2.5-Coder 72B | Q4 | ~36 GB | ~2.5 GB | ~39 GB |
| Qwen2.5-Coder 72B | FP8 | ~72 GB | ~2.5 GB | ~75 GB |
| Qwen2.5-Coder 32B | Q4 | ~18 GB | ~1.5 GB | ~20 GB |
| Llama 3.1 70B | Q4 | ~34 GB | ~2.5 GB | ~37 GB |
| DeepSeek-R1 Distill 32B | Q4 | ~20 GB | ~1.5 GB | ~22 GB |
| Qwen2.5-VL-7B (vision) | Q4 | ~4 GB | ~1 GB | ~5 GB |

### 3.3 Qwen2.5-Coder 72B (Detailed)

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | Total Estimate |
|---|---|---|---|---|
| **Q4_K_M** | ~36 GB | ~2.5 GB | ~10 GB | **~39-46 GB** |
| **Q8** | ~72 GB | ~2.5 GB | ~10 GB | **~75-82 GB** |
| **FP16** | ~144 GB | ~2.5 GB | ~10 GB | **~147-154 GB** |

### 3.4 Llama 3.1 70B (Detailed)

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | + KV Cache (128K ctx) | Total Estimate |
|---|---|---|---|---|---|
| **Q4_K_M** | ~34 GB | ~2.5 GB | ~10 GB | ~40 GB | **~37-74 GB** |
| **Q8** | ~69 GB | ~2.5 GB | ~10 GB | ~40 GB | **~72-109 GB** |
| **FP16** | ~138 GB | ~2.5 GB | ~10 GB | ~40 GB | **~141-178 GB** |

**Important:** Llama 3.1's 128K native context window can consume enormous KV cache memory. At 128K context, KV cache alone is ~40GB in FP16. Use GQA and KV cache quantization to reduce this.

### 3.5 DeepSeek-V3 / DeepSeek-R1 (Full)

These are 671B MoE models (37B active parameters per token). NOT feasible for local consumer hardware.

| Quantization | Model Weights | Minimum VRAM | Recommended Setup |
|---|---|---|---|
| **FP8** | ~685 GB | ~750 GB | 8x H100 80GB |
| **Q4 (AWQ)** | ~386 GB | ~420 GB | 8x A100 80GB or 5x H100 |
| **1.58-bit (Unsloth)** | ~131 GB | ~160 GB | 2x H100 80GB |

### 3.6 DeepSeek-Coder-V2

| Variant | Parameters (Total / Active) | FP16 VRAM | Q4 Estimate | Q8 Estimate |
|---|---|---|---|---|
| **Lite** | 16B / 2.4B active | ~32 GB | ~8-10 GB | ~16-18 GB |
| **Full** | 236B / 21B active | ~480 GB | ~120 GB | ~240 GB |

### 3.7 Vision Models for Screenshot QA

| Model | Parameters | VRAM (Q4/INT4) | DocVQA Score | Best For |
|---|---|---|---|---|
| **Qwen2.5-VL-7B** | 7B | ~4-5 GB | High | Best overall quality/size ratio |
| **Qwen3-VL-8B** | 8B | ~5-6 GB | Very High | Latest generation, best quality |
| **Llama 3.2 Vision 11B** | 11B | ~7-8 GB | 88.4% | Good all-around |
| **Qwen2.5-VL-3B** | 3B | ~2-3 GB | Good | Ultra-lightweight |
| **Qwen3-VL-2B** | 2B | ~1.5-2 GB | Decent | Minimal footprint |
| **PaddleOCR-VL-0.9B** | 0.9B | <1 GB | Good (OCR-specific) | Pure OCR extraction |

**Recommendation:** Qwen2.5-VL-7B or Qwen3-VL-8B provides excellent screenshot QA quality in ~5-6GB (Q4), easily fitting alongside a 70B coding model.

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

**For multi-user / multi-agent serving (NVIDIA):**
1. **vLLM** or **SGLang** — Best choices. SGLang often matches or exceeds vLLM throughput and handles concurrent requests more consistently.
2. **Ollama** — Good for development and simple setups. Becomes a bottleneck at high concurrency.
3. **llama.cpp** — Best for single-user, maximum-latency-optimized workloads.

**Multi-GPU tensor parallelism:**
- **vLLM:** Full tensor parallelism + pipeline parallelism. Set `tensor_parallel_size=N`. Requires NVIDIA GPUs.
- **SGLang:** Tensor parallelism + data parallelism. Strong multi-GPU scaling.
- **llama.cpp:** Only layer splitting (sequential, not parallel). Works, but 30-50% slower than true tensor parallel.

**For Apple Silicon:**
- **MLX is 30-50% faster** than llama.cpp Metal. Use MLX-based tools (mlx-lm, vllm-mlx, or LM Studio with MLX backend).
- **vllm-mlx**: Brings continuous batching to Apple Silicon. 21-87% higher throughput than llama.cpp. Supports OpenAI-compatible API, vision models.

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

## 5. Concurrent Inference Strategy

### Config #1 — Single PRO 6000 with MoE models (recommended)

```
RTX PRO 6000 (96 GB):
├── vLLM/SGLang: GPT-OSS-120B Q4 (~65 GB) — main agent @ ~134 tok/s
├── vLLM/SGLang: Qwen3-Coder-30B-A3B Q4 (~18 GB) — sub-agents @ ~150-200 tok/s
├── vLLM/SGLang: Qwen2.5-VL-7B Q4 (~5 GB) — vision/QA
└── ~8 GB free for KV cache

Routing: LiteLLM or nginx reverse proxy in front
```

### Config #2 — PRO 6000 + RTX 5090 with GPU separation

```
GPU 0 (PRO 6000, 96 GB):
├── GPT-OSS-120B Q4 (~65 GB) or Qwen2.5-Coder 72B Q4 (~39 GB) — main agent
├── Qwen2.5-VL-7B Q4 (~5 GB) — vision/QA
└── Free VRAM for KV cache

GPU 1 (RTX 5090, 32 GB):
├── Qwen3-Coder-30B-A3B Q4 (~18 GB) — sub-agents @ ~150-200 tok/s
└── ~14 GB free for KV cache

No bandwidth contention between main agent and sub-agents.
```

### Config #3 — DGX Spark 2+1 deployment

```
Units 1+2 (TP=2, 256 GB combined):
├── GPT-OSS-120B Q4 (~65 GB) — main agent @ ~75 tok/s
└── Ample KV cache headroom

Unit 3 (standalone, 128 GB):
├── Qwen3-Coder-30B-A3B Q4 (~18 GB) — sub-agents @ ~50-55 tok/s
└── Zero contention with main agent
```

---

## Sources

### GPU Benchmarks & Rankings
- [Hardware Corner — Definitive GPU Ranking for LLMs](https://www.hardware-corner.net/gpu-ranking-local-llm/)
- [Hardware Corner — RTX 5090 LLM Benchmarks](https://www.hardware-corner.net/rtx-5090-llm-benchmarks/)
- [CloudRift — RTX 4090 vs 5090 vs PRO 6000 Benchmark](https://www.cloudrift.ai/blog/benchmarking-rtx-gpus-for-llm-inference)
- [CloudRift — Benchmarking GPU Servers for LLM Inference](https://www.cloudrift.ai/blog/benchmarking-gpu-servers-for-llm-inference)
- [DatabaseMart — RTX PRO 6000 Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000)
- [DatabaseMart — 2x RTX 5090 Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2)
- [DatabaseMart — RTX 5090 Ollama Benchmark (single)](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090)
- [DatabaseMart — 4x A6000 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-a6000-4)
- [DatabaseMart — RTX 4090 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-rtx4090)
- [DatabaseMart — Dual A100 Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-dual-a100)
- [RunPod — RTX 5090 LLM Benchmarks](https://www.runpod.io/blog/rtx-5090-llm-benchmarks)
- [XiongjieDai — GPU Benchmarks on LLM Inference (GitHub)](https://github.com/XiongjieDai/GPU-Benchmarks-on-LLM-Inference)

### Apple Silicon
- [Olares Blog — Hardware Performance Benchmarking](https://blog.olares.com/local-ai-hardware-performance-benchmarking/)
- [Creative Strategies — Mac Studio M3 Ultra AI Workstation Review](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/)
- [llama.cpp Discussion #4167 — Apple Silicon Performance](https://github.com/ggml-org/llama.cpp/discussions/4167)
- [arXiv:2601.19139 — vllm-mlx: Native LLM Inference at Scale on Apple Silicon](https://arxiv.org/html/2601.19139v1)
- [arXiv:2511.05502 — Comparative Study: MLX, Ollama, llama.cpp on Apple Silicon](https://arxiv.org/abs/2511.05502)

### DGX Spark
- [LMSYS — DGX Spark In-Depth Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)
- [LMSYS — Optimizing GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/)
- [NVIDIA — DGX Spark Performance Blog](https://developer.nvidia.com/blog/how-nvidia-dgx-sparks-performance-enables-intensive-ai-tasks/)

### Inference Engines
- [Red Hat — vLLM or llama.cpp](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case)
- [Red Hat — Ollama vs vLLM Performance](https://developers.redhat.com/articles/2025/08/08/ollama-vs-vllm-deep-dive-performance-benchmarking)
- [vLLM Blog — v0.6.0 Performance Update](https://blog.vllm.ai/2024/09/05/perf-update.html)
- [vLLM Blog — Blackwell InferenceMAX](https://blog.vllm.ai/2025/10/09/blackwell-inferencemax.html)
- [vLLM Docs — Parallelism and Scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)
- [GPU-Mart — SGLang vs vLLM](https://www.gpu-mart.com/blog/sglang-vs-vllm)
- [Clarifai — SGLang vs vLLM vs TensorRT-LLM](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b)
- [BentoML — Benchmarking LLM Inference Backends](https://www.bentoml.com/blog/benchmarking-llm-inference-backends)

### Model VRAM Requirements
- [APXML — GPU Requirements for Qwen Models](https://apxml.com/posts/gpu-system-requirements-qwen-models)
- [APXML — GPU Requirements for Llama 3 Models](https://apxml.com/posts/ultimate-system-requirements-llama-3-models)
- [APXML — GPU Requirements for DeepSeek Models](https://apxml.com/posts/system-requirements-deepseek-models)
- [LocalLLM.in — Ollama VRAM Requirements Guide](https://localllm.in/blog/ollama-vram-requirements-for-local-llms)
- [Unsloth — DeepSeek-R1 Dynamic 1.58-bit](https://unsloth.ai/blog/deepseekr1-dynamic)

### Vision Models
- [Hugging Face — Qwen2.5-VL-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- [QwenLM — Qwen3-VL (GitHub)](https://github.com/QwenLM/Qwen3-VL)
- [Modal — 8 Top Open-Source OCR Models Compared](https://modal.com/blog/8-top-open-source-ocr-models-compared)

### Concurrent Inference & Multi-Model
- [NVIDIA — LLM Inference Benchmarking Fundamental Concepts](https://developer.nvidia.com/blog/llm-benchmarking-fundamental-concepts/)
- [Ferrari — llama.cpp Parallelism on A40 GPUs](https://medium.com/@ferraricorneloup.teo/how-many-developers-can-one-gpu-serve-benchmarking-llama-cpp-parallelism-on-a40-gpus-0ea2a8c36045)
- [Ollama GitHub — Multi-GPU Issues](https://github.com/ollama/ollama/issues/7104)
