# DGX Spark / ASUS Ascent — Research Deep-Dive

> Research date: February 2026. Extracted from the broader [alternative platforms research](../alternative-platforms-research.md).

---

## NVIDIA DGX Spark

| Spec | Value |
|------|-------|
| **Chip** | Grace Blackwell GB10 (ARM CPU + Blackwell GPU, NVLink-C2C) |
| **Memory** | 128 GB unified LPDDR5X |
| **Memory Bandwidth** | ~273 GB/s |
| **Compute** | 1 PFLOP FP4 AI |
| **TDP** | <240W |
| **Storage** | 4 TB NVMe SSD |
| **Connectivity** | 10GbE, Wi-Fi 7, ConnectX-7 (200 Gb/s), 2-unit clustering |
| **Price** | $3,999 |
| **Availability** | Available now (October 2025 launch). Stock constrained outside US. |
| **Key Differentiator** | Desktop AI supercomputer. Turnkey with DGX OS. CUDA ecosystem. |

**Assessment for our use case:** The DGX Spark is the benchmark for desktop AI appliances. 128 GB unified memory is attractive, but 273 GB/s bandwidth is severely limiting — roughly 7x slower than RTX PRO 6000. Estimated 70B Q4 inference: ~7-8 tok/s. Good for experimentation and smaller models, not for production 60-80 tok/s coding workflows. **Too slow for our performance targets on dense models, but meets targets on MoE models with 2-unit clustering (75 tok/s).**

Sources: [NVIDIA DGX Spark](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/), [DGX Spark Prices](https://www.glukhov.org/post/2025/10/nvidia-dgx-spark-prices/)

---

## ASUS Ascent GX10 Variants

All variants use the **identical GB10 Superchip** with 128 GB memory. Only storage/branding differs.

| Model | SSD | Price (CHF) | Link |
|-------|-----|-------------|------|
| **ASUS Ascent GX10** (cheapest) | 1 TB | **2,477** | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) |
| ASUS Ascent GX10 | 2 TB | 2,908 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000U0-p824150) |
| ASUS Ascent GX10 | 4 TB | 3,285 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000V0-p824154) |
| NVIDIA DGX Spark FE | 4 TB | 3,503 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) |

**Recommendation:** Buy the **ASUS Ascent 1 TB** for clusters. Same GPU, same memory, same performance. For a 3-unit cluster, ASUS saves **CHF 3,077** vs NVIDIA FE.

---

## Performance Characteristics

### Strengths
- **128 GB unified memory per unit** — fits the largest MoE models without quantization pressure
- **Near-silent operation** — <240W TDP, fanless or near-fanless
- **Turnkey** — no assembly, no driver configuration, DGX OS pre-installed
- **Scalable** — add units incrementally at ~CHF 2,477 each
- **CUDA ecosystem** — full compatibility with vLLM, SGLang, TRT-LLM, Ollama

### Limitations
- **273 GB/s bandwidth** — 7x slower than RTX PRO 6000 (1,792 GB/s). This is the fundamental constraint.
- **Dense 70B models unusable** — ~5 tok/s on Llama 3.1 70B (FP8). Only MoE models are viable.
- **ARM platform** — most frameworks work, but some Python packages lack ARM wheels
- **Multi-node complexity** — clustering requires NCCL configuration, passwordless SSH, Docker containers

### Clustering Performance
- **2 units (TP=2):** ~1.4x speedup (not 2x) due to 200 Gbps interconnect overhead
- **Direct connection:** QSFP56 DAC cable for 2 units, Ethernet switch for 3+
- **Software:** NCCL for GPU collectives, supported by vLLM, SGLang, TensorRT-LLM

---

## Sources

- [NVIDIA DGX Spark](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/)
- [NVIDIA DGX Spark Performance Blog](https://developer.nvidia.com/blog/how-nvidia-dgx-sparks-performance-enables-intensive-ai-tasks/)
- [LMSYS — DGX Spark In-Depth Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)
- [LMSYS — Optimizing GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/)
- [DGX Spark Clustering Guide](https://docs.nvidia.com/dgx/dgx-spark/spark-clustering.html)
- [DGX Spark Prices](https://www.glukhov.org/post/2025/10/nvidia-dgx-spark-prices/)
- [DGX Station Blog](https://blogs.nvidia.com/blog/dgx-spark-and-station-open-source-frontier-models/)
- [DGX Spark Alternatives](https://research.aimultiple.com/dgx-spark-alternatives/)
