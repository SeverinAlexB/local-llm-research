# NVIDIA GPU Benchmarks

> Research compiled February 2026. All figures are approximate and depend on context length, batch size, quantization method, driver version, and inference software.

For shared reference data (memory requirements, inference software comparison), see [benchmarks.md](../benchmarks.md).

---

## 1. Token Generation Speed

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

### 1.3 MoE Coding Models on NVIDIA Hardware

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

## 3. Concurrent Inference Strategy

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

### Concurrent Inference & Multi-Model
- [NVIDIA — LLM Inference Benchmarking Fundamental Concepts](https://developer.nvidia.com/blog/llm-benchmarking-fundamental-concepts/)
- [Ferrari — llama.cpp Parallelism on A40 GPUs](https://medium.com/@ferraricorneloup.teo/how-many-developers-can-one-gpu-serve-benchmarking-llama-cpp-parallelism-on-a40-gpus-0ea2a8c36045)
- [Ollama GitHub — Multi-GPU Issues](https://github.com/ollama/ollama/issues/7104)
