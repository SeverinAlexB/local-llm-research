# Apple Silicon Benchmarks

> Research compiled February 2026. All figures are approximate and depend on context length, batch size, quantization method, and inference software.

For shared reference data (memory requirements, inference software comparison), see [benchmarks.md](../benchmarks.md).

---

## 1. Token Generation Speed

### 1.1 70B-Class Models

All figures are single-stream token generation (tok/s) unless noted. Context ~4K tokens.

| Hardware | Q4_K_M (~4-bit) | Q8 (~8-bit) | FP16 | Notes |
|---|---|---|---|---|
| **Mac Studio M3 Ultra** (192GB) | **~15-20 tok/s** | ~10-14 | ~6-8 | ~819 GB/s bandwidth. MLX achieves ~30-50% faster than llama.cpp |

### 1.2 7B-32B Models (Sub-Agent Workloads)

Single-stream token generation (tok/s), context ~4K tokens.

| Hardware | 7B Q4 | 7B Q8/FP16 | 14B Q4 | 32B Q4 | Notes |
|---|---|---|---|---|---|
| **Mac M3 Ultra** (192GB) | ~120-130 (llama.cpp) / ~240+ (MLX) | ~90-110 | ~70-90 | ~35-50 | MLX significantly faster than llama.cpp |

### 1.3 MoE Coding Models on Apple Silicon

**Mac Studio M3 Ultra (256 GB, Olares benchmark suite):**

| Model | Type | Total/Active Params | 1 concurrent | 8 concurrent |
|---|---|---|---|---|
| **GPT-OSS-120B** | MoE | 120B / ~20B | **69.4 tok/s** | 19.1 tok/s |
| **Qwen3-30B-A3B** | MoE | 30B / 3.3B | **84 tok/s** | 24.9 tok/s |

**Key Insight:** MoE models make the 60-80 tok/s target achievable on Apple Silicon. Dense 70B models at ~15-20 tok/s are now the fallback, not the primary workload.

Sources: [Olares Blog Benchmarks](https://blog.olares.com/local-ai-hardware-performance-benchmarking/), [LMSYS DGX Spark Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)

---

## 2. Concurrent Inference

### 2.1 Concurrent Scaling (Olares Blog Benchmarks)

Tested on Mac Studio M3 Ultra with the Olares benchmark suite:

| Model | 1 concurrent | 8 concurrent |
|---|---|---|
| Qwen3-30B-A3B | 84 tok/s | 24.9 tok/s |
| GPT-OSS-120B | 69.4 tok/s | 19.1 tok/s |

On M4 Max:

| Model | 1 concurrent | 8 concurrent |
|---|---|---|
| Qwen3-30B-A3B | 81.2 tok/s | 19.9 tok/s |

### 2.2 Concurrency Software on Apple Silicon

- **MLX (raw framework):** Single-stream optimized; not thread-safe for concurrent Metal inference. Use separate processes as workaround.
- **vllm-mlx (recommended):** Provides continuous batching on Apple Silicon. Scales to ~4.3x aggregate throughput at 16 concurrent requests. Supports OpenAI-compatible API, vision models.
- **MLC-LLM:** Best option for multi-worker HTTP serving with micro-batching.

---

## Sources

### Apple Silicon
- [Olares Blog — Hardware Performance Benchmarking](https://blog.olares.com/local-ai-hardware-performance-benchmarking/)
- [Creative Strategies — Mac Studio M3 Ultra AI Workstation Review](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/)
- [llama.cpp Discussion #4167 — Apple Silicon Performance](https://github.com/ggml-org/llama.cpp/discussions/4167)
- [arXiv:2601.19139 — vllm-mlx: Native LLM Inference at Scale on Apple Silicon](https://arxiv.org/html/2601.19139v1)
- [arXiv:2511.05502 — Comparative Study: MLX, Ollama, llama.cpp on Apple Silicon](https://arxiv.org/abs/2511.05502)
