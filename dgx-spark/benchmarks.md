# DGX Spark / ASUS Ascent Benchmarks

> Research compiled February 2026. All figures are approximate and depend on context length, batch size, quantization method, and inference software.

For shared reference data (memory requirements, inference software comparison), see [benchmarks.md](../benchmarks.md).

---

## 1. Token Generation Speed

### 1.1 MoE and Dense Models on DGX Spark

**DGX Spark / ASUS Ascent (GB10, 128 GB per unit):**

| Model | Type | 1 unit tok/s | 2 units (TP=2) tok/s |
|---|---|---|---|
| **GPT-OSS-120B** | MoE | 50-55 | **75** |
| **GPT-OSS-20B** | MoE | 50 | — |
| **Llama 3.1 70B** | Dense (FP8) | 2.7 | ~5 |

**Key Insight:** The DGX Spark is effective for MoE models (75 tok/s on GPT-OSS-120B with 2 units) but unusable for dense 70B models (~5 tok/s). The 273 GB/s bandwidth per unit is the limiting factor for dense models.

Sources: [LMSYS DGX Spark Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/), [LMSYS GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/)

### 1.2 Qwen3.5-397B-A17B (Released Feb 16, 2026)

397B total / **17B active** per token. IQ4_XS ~212 GB, Q4_K_M ~241 GB.

**Fits on a 2-unit cluster (256 GB)** at IQ4_XS with ~40 GB headroom. With 17B active params at Q4 (~8.5 GB read per token) and ~546 GB/s combined bandwidth (2 units):

KV cache is exceptionally small due to the hybrid DeltaNet architecture (only 15/60 layers use standard attention, 2 KV heads each). At 128K context: ~3.9 GB. At 262K: ~7.7 GB.

| Config | Quant | Total VRAM (128K ctx) | Est. tok/s | Notes |
|---|---|---|---|---|
| 2 units (256 GB, TP=2) | IQ4_XS (~212 GB) | ~216 GB | **~40-50** | Fits with 128K context, ~40 GB free for sub-agent |
| 3 units (384 GB, TP=3) | Q4_K_M (~241 GB) | ~245 GB | **~55-70** | Comfortable fit, room for 262K context + sub-agents |

**Practical note:** The tiny KV cache makes the 2-unit cluster more viable than expected — even at 128K context, only ~216 GB is needed. The 3-unit cluster remains compelling for Q4_K_M quality and concurrent sub-agents. Day-one software support is unoptimized; expect numbers to improve.

Sources: [HuggingFace — Qwen3.5-397B-A17B-GGUF](https://huggingface.co/unsloth/Qwen3.5-397B-A17B-GGUF)

---

## 2. Concurrent Inference Strategy

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

### DGX Spark
- [LMSYS — DGX Spark In-Depth Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)
- [LMSYS — Optimizing GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/)
- [NVIDIA — DGX Spark Performance Blog](https://developer.nvidia.com/blog/how-nvidia-dgx-sparks-performance-enables-intensive-ai-tasks/)
