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
