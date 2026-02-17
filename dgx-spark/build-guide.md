# DGX Spark / ASUS Ascent — Build Guide & Cluster Configurations

> Last updated: February 17, 2026. All prices verified against toppreise.ch.

For the rationale behind this configuration, see [decision-summary.md](../decision-summary.md). For DGX Spark deep-dive, see [research.md](research.md).

---

## Config #2b: DGX Spark / ASUS Ascent Cluster

Turnkey mini-PCs with NVIDIA GB10 Grace Blackwell Superchip. No assembly required. Each unit: 128 GB unified LPDDR5x, 273 GB/s bandwidth, ~140W TDP, near-silent.

### 2-Unit Cluster (~CHF 5,004)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **Unit 1** | [ASUS Ascent GX10-GG0003BN (1 TB, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 2 | **Unit 2** | [ASUS Ascent GX10-GG0003BN (1 TB, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 3 | **Cable** | QSFP56 200 Gbps DAC cable (0.4-1m) | ~50 |
| | **TOTAL** | | **~CHF 5,004** |

**Memory:** 256 GB. **Performance (TP=2):** ~75 tok/s on GPT-OSS-120B. **Power:** ~280W. **Noise:** Near silent.

### 3-Unit Cluster (~CHF 7,481)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1-3 | **3x ASUS Ascent GX10-GG0003BN** | 1 TB each, 128 GB each | 3 x 2,477 = 7,431 |
| 4 | **Cable** | QSFP56 200 Gbps DAC cable | ~50 |
| | **TOTAL** | | **~CHF 7,481** |

**Memory:** 384 GB. **Recommended deployment:** Units 1+2 paired (TP=2) → 75 tok/s main agent. Unit 3 standalone → 50-55 tok/s sub-agent. Zero contention.

### Spark Stacking — How It Works

- **2 units:** Direct QSFP56 DAC cable (200 Gbps). No switch needed.
- **3+ units:** Connect via a 200G/400G Ethernet switch.
- Software: NCCL for GPU collectives, supported by vLLM, SGLang, TensorRT-LLM.
- Scaling: ~1.4x for 2 units (not 2x) due to 200 Gbps interconnect overhead.

### ASUS Ascent vs NVIDIA DGX Spark FE

All variants use the **identical GB10 Superchip** with 128 GB memory. Only storage/branding differs.

| Model | SSD | Price (CHF) | Link |
|-------|-----|-------------|------|
| **ASUS Ascent GX10** (cheapest) | 1 TB | **2,477** | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) |
| ASUS Ascent GX10 | 2 TB | 2,908 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000U0-p824150) |
| ASUS Ascent GX10 | 4 TB | 3,285 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000V0-p824154) |
| NVIDIA DGX Spark FE | 4 TB | 3,503 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) |

**Recommendation:** Buy the **ASUS Ascent 1 TB** for clusters. Same GPU, same memory, same performance. For a 3-unit cluster, ASUS saves **CHF 3,077** vs NVIDIA FE.

### DGX Spark vs RTX PRO 6000

| | 2x ASUS Ascent | RTX PRO 6000 (AM5) |
|--|----------------|-------------------|
| **Price** | **CHF 5,004** | CHF 9,485 |
| **Memory** | **256 GB** | 96 GB |
| **MoE main agent** | 75 tok/s (TP=2) | **134 tok/s** |
| **Dense 70B** | ~5 tok/s (unusable) | **32 tok/s** |
| **Noise** | **Near silent** | Quiet |
| **Setup** | **Turnkey** | Build required |
| **200K context** | Easy (128 GB/unit) | Tight (INT8 KV required) |

The PRO 6000 is **1.8x faster** on MoE models due to 6.6x higher bandwidth. The Spark cluster's advantages: half the price, near-silent, 256-384 GB memory, turnkey.

### Practical Considerations

- **ARM platform:** GB10 uses ARM (Grace) CPUs. Most inference frameworks work (vLLM, SGLang, TRT-LLM, Ollama). Some Python packages may lack ARM wheels.
- **Multi-node setup:** Requires NCCL configuration, passwordless SSH between nodes, Docker containers. More complex than single-GPU `ollama run`.
- **Storage:** 1 TB holds 15-30 models in Q4. External NVMe over USB4 supported.
- **Upgrade path:** Add units incrementally at ~CHF 2,477 each. Far cheaper than adding a 2nd PRO 6000 (~CHF 7,691).

Sources: [NVIDIA DGX Spark](https://www.nvidia.com/en-us/products/workstations/dgx-spark/), [NVIDIA DGX Spark Performance Blog](https://developer.nvidia.com/blog/how-nvidia-dgx-sparks-performance-enables-intensive-ai-tasks/), [LMSYS DGX Spark Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/), [LMSYS GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/), [DGX Spark Clustering Guide](https://docs.nvidia.com/dgx/dgx-spark/spark-clustering.html)
