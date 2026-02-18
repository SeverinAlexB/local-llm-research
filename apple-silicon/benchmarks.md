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

### 1.4 Qwen3.5-397B-A17B on Apple Silicon (Released Feb 16, 2026)

The largest open-weight MoE model from Alibaba. 397B total parameters, **17B active per token**, 512 experts (11 active). Native multimodal (text + vision + video). Claims competitive with GPT-5.2 / Claude Opus 4.5 on 80% of benchmarks (self-reported, pending independent verification).

**VRAM requirements:**

KV cache is **exceptionally small** thanks to the hybrid DeltaNet architecture: only 15 of 60 layers use standard attention (2 KV heads each), the other 45 layers use Gated DeltaNet with a fixed ~22-90 MB recurrent state. At 8K context: ~0.3 GB. At 128K: ~3.9 GB. At 262K (max): ~7.7 GB.

| Quantization | Model Size | + KV (128K ctx) | Total (128K) | Fits 256 GB? | Fits 512 GB? |
|---|---|---|---|---|---|
| IQ2_XXS (2-bit) | ~130 GB | ~3.9 GB | ~134 GB | Yes | Yes |
| IQ4_XS (4-bit) | ~212 GB | ~3.9 GB | ~216 GB | Yes (~40 GB free) | Yes |
| Q4_K_M (4-bit) | ~241 GB | ~3.9 GB | ~245 GB | Tight (~11 GB free) | Yes |
| Q8_0 (8-bit) | ~422 GB | ~3.9 GB | ~426 GB | No | Barely |

**Estimated performance on M3 Ultra (819 GB/s):**

With 17B active params at Q4 (~8.5 GB active weights per token), the bandwidth math predicts ~96 tok/s theoretical, ~62-72 tok/s real — comparable to GPT-OSS-120B (69 tok/s with 20B active). Early day-one reports show lower numbers due to unoptimized software support for the novel Gated DeltaNet architecture.

| Config | Quant | Est. tok/s | Notes |
|---|---|---|---|
| M3 Ultra 256 GB | IQ4_XS (~212 GB) | **~50-70** (optimized) | Fits with 128K context (~216 GB total). ~40 GB free for OS + sub-agent |
| M3 Ultra 512 GB | Q4_K_M (~241 GB) | **~60-75** (optimized) | Comfortable, room for 262K context + sub-agents |
| M3 Ultra 512 GB | Q8_0 (~422 GB) | **~30-40** (optimized) | Near-lossless quality, barely fits |

**Day-one caveat:** MLX/llama.cpp support is brand new. Expect ~15-30 tok/s initially, converging to the estimates above as MoE routing optimizations land. One HN user reported 20+ tok/s on MXFP4 (~216 GB) on an unspecified MacBook.

**Practical assessment for 256 GB Mac Studio:** The low KV cache makes this more viable than initially expected. At IQ4_XS (~212 GB) with 128K context (~216 GB total), ~40 GB remains for OS and potentially a small sub-agent model (e.g. Qwen3-30B-A3B Q4 at ~18 GB would be very tight). For full multi-agent workflows with multiple concurrent models, the proven **Qwen3-Next-80B-A3B** (60-70 tok/s, ~43 GB) and **Qwen3-30B-A3B** (84 tok/s, ~18 GB) still leave more headroom. On the 512 GB config, Qwen3.5-397B fits comfortably alongside sub-agents.

Sources: [MarkTechPost — Qwen3.5 Release](https://www.marktechpost.com/2026/02/16/alibaba-qwen-team-releases-qwen3-5-397b-moe-model-with-17b-active-parameters-and-1m-token-context-for-ai-agents/), [HuggingFace — Qwen3.5-397B-A17B-GGUF](https://huggingface.co/unsloth/Qwen3.5-397B-A17B-GGUF), [Hacker News Discussion](https://news.ycombinator.com/item?id=47032876)

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
