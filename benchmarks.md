# AI Inference Hardware — Shared Benchmarks & Technical Reference

> Research compiled February 2026. All figures are approximate and depend heavily on context length, batch size, quantization method, driver version, and inference software.

For **model quality** benchmarks (SWE-Bench, LiveCodeBench, Aider — how good are these models at coding?), see [coding-model-benchmarks.md](coding-model-benchmarks.md).

For platform-specific benchmark data, see:
- [nvidia-gpu/benchmarks.md](nvidia-gpu/benchmarks.md) — NVIDIA GPU token generation speeds, concurrent inference, deployment strategies
- [apple-silicon/benchmarks.md](apple-silicon/benchmarks.md) — Apple Silicon token generation speeds, concurrent scaling
- [dgx-spark/benchmarks.md](dgx-spark/benchmarks.md) — DGX Spark / ASUS Ascent performance data

---

## Table of Contents

1. [Memory Requirements for Specific Models](#1-memory-requirements-for-specific-models)
2. [Inference Software Comparison](#2-inference-software-comparison)
3. [Sources](#sources)

---

## 1. Memory Requirements for Specific Models

### 1.1 MoE Models

| Model | Total / Active Params | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|---|
| GPT-OSS-120B | 120B / ~20B | Q4 | ~65 GB | ~2 GB | ~67 GB |
| Qwen3-Coder-30B-A3B | 30B / 3.3B | Q4 | ~18 GB | ~1 GB | ~19 GB |
| MiniMax-M2.5 | 230B / 10B | NVFP4 | ~130 GB | ~5 GB | ~135 GB |
| Qwen3.5-397B-A17B | 397B / 17B | IQ4_XS | ~212 GB | ~0.3 GB (8K) | ~212 GB |
| Qwen3.5-397B-A17B | 397B / 17B | Q4_K_M | ~241 GB | ~0.3 GB (8K) | ~241 GB |
| Qwen3-Next-80B-A3B | 80B / ~3B | Q4 | ~43 GB | ~1 GB | ~44 GB |
| DeepSeek-Coder-V2-Lite | 16B / 2.4B | Q4 | ~10 GB | ~1 GB | ~11 GB |

### 1.2 Dense Models

| Model | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|
| Qwen2.5-Coder 72B | Q4 | ~36 GB | ~2.5 GB | ~39 GB |
| Qwen2.5-Coder 72B | FP8 | ~72 GB | ~2.5 GB | ~75 GB |
| Qwen2.5-Coder 32B | Q4 | ~18 GB | ~1.5 GB | ~20 GB |
| Llama 3.1 70B | Q4 | ~34 GB | ~2.5 GB | ~37 GB |
| DeepSeek-R1 Distill 32B | Q4 | ~20 GB | ~1.5 GB | ~22 GB |
| Qwen2.5-VL-7B (vision) | Q4 | ~4 GB | ~1 GB | ~5 GB |

### 1.3 Qwen2.5-Coder 72B (Detailed)

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | Total Estimate |
|---|---|---|---|---|
| **Q4_K_M** | ~36 GB | ~2.5 GB | ~10 GB | **~39-46 GB** |
| **Q8** | ~72 GB | ~2.5 GB | ~10 GB | **~75-82 GB** |
| **FP16** | ~144 GB | ~2.5 GB | ~10 GB | **~147-154 GB** |

### 1.4 Llama 3.1 70B (Detailed)

| Quantization | Model Weights | + KV Cache (8K ctx) | + KV Cache (32K ctx) | + KV Cache (128K ctx) | Total Estimate |
|---|---|---|---|---|---|
| **Q4_K_M** | ~34 GB | ~2.5 GB | ~10 GB | ~40 GB | **~37-74 GB** |
| **Q8** | ~69 GB | ~2.5 GB | ~10 GB | ~40 GB | **~72-109 GB** |
| **FP16** | ~138 GB | ~2.5 GB | ~10 GB | ~40 GB | **~141-178 GB** |

**Important:** Llama 3.1's 128K native context window can consume enormous KV cache memory. At 128K context, KV cache alone is ~40GB in FP16. Use GQA and KV cache quantization to reduce this.

### 1.5 DeepSeek-V3 / DeepSeek-R1 (Full)

These are 671B MoE models (37B active parameters per token). NOT feasible for local consumer hardware.

| Quantization | Model Weights | Minimum VRAM | Recommended Setup |
|---|---|---|---|
| **FP8** | ~685 GB | ~750 GB | 8x H100 80GB |
| **Q4 (AWQ)** | ~386 GB | ~420 GB | 8x A100 80GB or 5x H100 |
| **1.58-bit (Unsloth)** | ~131 GB | ~160 GB | 2x H100 80GB |

### 1.6 DeepSeek-Coder-V2

| Variant | Parameters (Total / Active) | FP16 VRAM | Q4 Estimate | Q8 Estimate |
|---|---|---|---|---|
| **Lite** | 16B / 2.4B active | ~32 GB | ~8-10 GB | ~16-18 GB |
| **Full** | 236B / 21B active | ~480 GB | ~120 GB | ~240 GB |

### 1.7 Vision Models for Screenshot QA

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

## 2. Inference Software Comparison

### 2.1 Feature Matrix

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

### 2.2 When to Use What

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

### 2.3 Performance Comparison: Throughput at Scale

Measured on similar hardware (A100-class), Llama 70B, high concurrency:

| Engine | Relative Throughput | Notes |
|---|---|---|
| vLLM | 1.0x (baseline) | PagedAttention, continuous batching |
| SGLang | 1.1-1.3x | RadixAttention; excels with shared prefixes (up to 158K tok/s with 75% cache hit) |
| TensorRT-LLM | 1.5-1.7x | Requires NVIDIA GPUs, complex setup, model-specific compilation |
| llama.cpp | 0.03-0.1x | Not optimized for high-concurrency GPU serving |
| Ollama | 0.05-0.1x | Built on llama.cpp; 41 TPS peak vs vLLM's 793 TPS |

---

## Sources

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
- [HuggingFace — Qwen3.5-397B-A17B-GGUF (Unsloth)](https://huggingface.co/unsloth/Qwen3.5-397B-A17B-GGUF)
- [APXML — GPU Requirements for Qwen Models](https://apxml.com/posts/gpu-system-requirements-qwen-models)
- [APXML — GPU Requirements for Llama 3 Models](https://apxml.com/posts/ultimate-system-requirements-llama-3-models)
- [APXML — GPU Requirements for DeepSeek Models](https://apxml.com/posts/system-requirements-deepseek-models)
- [LocalLLM.in — Ollama VRAM Requirements Guide](https://localllm.in/blog/ollama-vram-requirements-for-local-llms)
- [Unsloth — DeepSeek-R1 Dynamic 1.58-bit](https://unsloth.ai/blog/deepseekr1-dynamic)

### Vision Models
- [Hugging Face — Qwen2.5-VL-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- [QwenLM — Qwen3-VL (GitHub)](https://github.com/QwenLM/Qwen3-VL)
- [Modal — 8 Top Open-Source OCR Models Compared](https://modal.com/blog/8-top-open-source-ocr-models-compared)
