# DGX Spark Cluster — Build Guide

> Last updated: February 17, 2026
> All prices verified against toppreise.ch on this date.

---

## Overview

A cluster of NVIDIA DGX Spark units (GB10 Grace Blackwell Superchip) as an alternative to the RTX PRO 6000 workstation build. Each unit is a turnkey mini-PC with 128 GB unified LPDDR5x memory and a Blackwell GPU — no assembly required.

**Key specs of each DGX Spark unit:**
- NVIDIA GB10 Grace Blackwell Superchip
- 128 GB unified LPDDR5x memory (shared between CPU and GPU)
- 273 GB/s memory bandwidth
- 1 PFLOPS FP4 AI compute (5th gen Tensor Cores)
- 20 ARM cores (10x Cortex-X925 + 10x Cortex-A725)
- 2x QSFP56 200 Gbps ports (ConnectX-7 SmartNIC) for inter-unit clustering
- Wi-Fi 7, Bluetooth 5.4, 10 GbE
- ~140W TDP
- Near-silent desktop form factor
- NVIDIA DGX OS (Ubuntu-based) or standard Ubuntu

**Performance (single unit, best framework):**

| Model | Type | tok/s (decode) | Framework |
|-------|------|----------------|-----------|
| GPT-OSS-120B | MoE (120B total / ~20B active) | **50-55** | SGLang / TRT-LLM |
| GPT-OSS-20B | MoE (20B total / ~5B active) | **50** | Ollama |
| Qwen 2.5 7B | Dense | 50 | Ollama |
| DeepSeek R1 14B | Dense | ~11 (batch 1) | SGLang |
| Llama 3.1 70B | Dense (FP8) | **2.7** | SGLang |

**Critical observation:** MoE models run at 50-55 tok/s (only active params read per token). Dense 70B at 2.7 tok/s is unusable — the 273 GB/s bandwidth cannot push 35+ GB of weights fast enough.

---

## Why Consider DGX Spark?

The RTX PRO 6000 is faster per-stream (134 tok/s vs 50-55 tok/s on MoE). The DGX Spark's advantages are different:

| Factor | DGX Spark (per unit) | RTX PRO 6000 Build |
|--------|---------------------|-------------------|
| Memory per unit | **128 GB** | 96 GB VRAM |
| Bandwidth | 273 GB/s | **1,792 GB/s** (6.6x) |
| Setup | **Turnkey** (plug & power) | PC build required |
| Noise | **Near silent** (~140W) | Quiet (~300W GPU) |
| Scalability | Add more units | Add 2nd GPU (NVLink) |
| Price per unit | **CHF 2,477-3,503** | CHF 9,485-10,666 (full build) |

---

## Spark Stacking — Connecting Units Together

Two or more DGX Spark units can be connected for tensor parallelism, splitting a model across units to improve speed.

### How it works

- **2 units:** Connect directly with a QSFP56 DAC cable (200 Gbps aggregate). No switch needed.
- **3+ units:** Connect via a 200G/400G Ethernet switch with QSFP56 DAC cables.
- Software: NCCL v2.28.3 for GPU collective operations, MPI for CPU communication.
- Supported by: vLLM, SGLang, TensorRT-LLM.

### Dual-unit performance (GPT-OSS-120B)

| Config | Framework | Decode tok/s | Speedup vs single |
|--------|-----------|-------------|-------------------|
| 1x DGX Spark | SGLang | 52 | — |
| **2x DGX Spark (TP=2)** | **vLLM** | **75** | **1.44x** |
| 2x DGX Spark (TP=2) | SGLang | ~65-70 | ~1.3x |

Scaling is ~1.4x (not 2x) because the 200 Gbps interconnect (25 GB/s) adds synchronization overhead at each transformer layer. Each layer requires an all-reduce operation across the network — fast enough to avoid being the primary bottleneck, but enough to prevent linear scaling.

### Larger models across 2 units

| Model | Decode tok/s | Notes |
|-------|-------------|-------|
| Qwen3 235B (NVFP4) | 11.7 | Requires >128 GB — cannot run on single unit |
| Llama 3.1 405B (FP4) | Supported | NVIDIA claims support; no public decode benchmarks |

The 256 GB combined memory enables models that physically cannot fit on a single 96 GB GPU card.

---

## Configurations

### Option A: 2x DGX Spark Cluster (~CHF 4,954 – 7,055)

The minimum cluster. Two units connected directly for tensor parallelism on a single model, or running independent models.

#### With ASUS Ascent (cheapest GB10 hardware)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **Unit 1** | [ASUS Ascent GX10-GG0003BN (1 TB SSD, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 2 | **Unit 2** | [ASUS Ascent GX10-GG0003BN (1 TB SSD, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 3 | **Cable** | QSFP56 200 Gbps DAC cable (0.4-1m) | ~50 |
| | **TOTAL** | | **~CHF 5,004** |

#### With NVIDIA DGX Spark Founders Edition

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **Unit 1** | [NVIDIA DGX Spark FE (4 TB SSD, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) | 3,503 |
| 2 | **Unit 2** | [NVIDIA DGX Spark FE (4 TB SSD, 128 GB)](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) | 3,503 |
| 3 | **Cable** | QSFP56 200 Gbps DAC cable (0.4-1m) | ~50 |
| | **TOTAL** | | **~CHF 7,055** |

**Memory:** 256 GB (2 × 128 GB)
**Performance (TP=2):** ~75 tok/s on GPT-OSS-120B (meets 60-80 tok/s target)
**Power:** ~280W total
**Noise:** Near silent

**Modes of operation:**
1. **Tensor parallelism (TP=2):** Both units serve one model together. ~75 tok/s on GPT-OSS-120B. Best for maximum single-stream speed.
2. **Independent models:** Each unit runs its own model. 50-55 tok/s per stream on MoE. Best for multi-agent workflows with zero contention.

---

### Option B: 3x DGX Spark Cluster (~CHF 7,431 – 10,558)

Three units allow a paired cluster for the main agent + a standalone unit for sub-agents.

#### With ASUS Ascent (cheapest)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **Unit 1** | [ASUS Ascent GX10-GG0003BN (1 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 2 | **Unit 2** | [ASUS Ascent GX10-GG0003BN (1 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 3 | **Unit 3** | [ASUS Ascent GX10-GG0003BN (1 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) | 2,477 |
| 4 | **Cable** | QSFP56 200 Gbps DAC cable (0.4-1m) | ~50 |
| | **TOTAL** | | **~CHF 7,481** |

#### With NVIDIA DGX Spark Founders Edition

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **Unit 1** | [NVIDIA DGX Spark FE (4 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) | 3,503 |
| 2 | **Unit 2** | [NVIDIA DGX Spark FE (4 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) | 3,503 |
| 3 | **Unit 3** | [NVIDIA DGX Spark FE (4 TB SSD)](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) | 3,503 |
| 4 | **Cable** | QSFP56 200 Gbps DAC cable (0.4-1m) | ~50 |
| | **TOTAL** | | **~CHF 10,558** |

**Memory:** 384 GB (3 × 128 GB)
**Power:** ~420W total
**Noise:** Near silent

**Recommended deployment: 2 paired + 1 independent**
- Units 1+2 connected via QSFP → **75 tok/s** main agent (TP=2, GPT-OSS-120B)
- Unit 3 standalone → **50-55 tok/s** sub-agent stream (independent MoE model)
- Zero bandwidth contention between main and sub-agent

**Alternative: 3-node TP cluster** (requires a 200G switch, ~CHF 200+)
- All 3 in tensor parallelism → estimated ~85-95 tok/s on GPT-OSS-120B
- 384 GB unified → can run 300B+ models
- Loses independent sub-agent stream

---

## ASUS Ascent vs NVIDIA DGX Spark FE

All variants use the **identical GB10 Superchip** with 128 GB unified memory. The only difference is storage, branding, and pre-installed OS.

| Model | SSD | OS | Price (CHF) | Toppreise |
|-------|-----|----|-------------|-----------|
| **ASUS Ascent GX10-GG0003BN** | 1 TB | Ubuntu | **2,477** | [link](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) |
| **ASUS Ascent GX10-GG0026BN** | 2 TB | Ubuntu | **2,908** | [link](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000U0-p824150) |
| **ASUS Ascent GX10-GG0026BN** | 4 TB | Ubuntu | **3,285** | [link](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000V0-p824154) |
| **NVIDIA DGX Spark FE** | 4 TB | DGX OS | **3,503** | [link](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) |

**Recommendation:** Buy the **ASUS Ascent** with the storage you need. The 1 TB model at CHF 2,477 saves CHF 1,026 per unit vs the Founders Edition — same GPU, same memory, same performance. You can always add external NVMe storage. DGX OS can be installed on any GB10 unit.

For a 3-unit cluster, ASUS Ascent 1 TB saves **CHF 3,077** vs NVIDIA FE (CHF 7,481 vs CHF 10,558).

---

## Comparison vs RTX PRO 6000 Builds

### Price-matched comparison

| | 3x ASUS Ascent | 2x DGX Spark FE | RTX PRO 6000 (AM5) | RTX PRO 6000 (TR) |
|--|----------------|-----------------|--------------------|--------------------|
| **Price** | **CHF 7,481** | **CHF 7,055** | **CHF 9,485** | **CHF 10,666** |
| **Total memory** | **384 GB** | **256 GB** | 96 GB | 96 GB |
| **MoE main agent** | 75 tok/s (TP=2) | 75 tok/s (TP=2) | **134 tok/s** | **134 tok/s** |
| **MoE sub-agent** | 50 tok/s (3rd unit) | — (both in TP) | shares GPU | shares GPU |
| **Dense 70B** | ~5 tok/s (TP=2) | ~5 tok/s (TP=2) | **32 tok/s** | **32 tok/s** |
| **Power** | 420W | 280W | 475W | 525W |
| **Noise** | **Near silent** | **Near silent** | Quiet | Quiet |
| **Setup** | Turnkey | Turnkey | Build required | Build required |
| **200K context** | Easy (128 GB/unit) | Easy (128 GB/unit) | Tight (INT8 KV req) | Tight (INT8 KV req) |
| **Upgrade** | Add units | Add units | NVLink 2nd GPU | NVLink 2nd GPU |

### Speed analysis

The RTX PRO 6000 is **1.8x faster** on MoE models (134 vs 75 tok/s) due to 6.6x higher memory bandwidth. This gap is fundamental — no software optimization can close it.

However, the DGX Spark cluster has advantages:
- **Memory**: 256-384 GB vs 96 GB. Models that exceed 96 GB (e.g., Qwen3-Coder-480B at ~240 GB) can only run on the Spark cluster.
- **Concurrency**: With 3 units (2+1 deployment), the main agent gets 75 tok/s while a sub-agent gets 50 tok/s with zero contention. The PRO 6000 shares bandwidth.
- **200K context**: Each Spark unit has 128 GB — KV cache at 200K in FP16 is no problem. The PRO 6000's 96 GB requires INT8 KV cache quantization.

### Against requirements

| Requirement | 2x Spark (TP=2) | 3x Spark (2+1) | RTX PRO 6000 | Target |
|---|---|---|---|---|
| Main agent speed | **75 tok/s** | **75 tok/s** | **134 tok/s** | 60-80 |
| Sub-agent speed | — | **50 tok/s** | ~150-200 tok/s | 15-20 |
| Dense 70B fallback | ~5 tok/s | ~5 tok/s | **32 tok/s** | 30 |
| Concurrent streams | 1 (TP=2 locked) | 2 (1 cluster + 1 solo) | 5-10 (shared) | 5-10 |

- **2x Spark cluster** meets the main agent target (75 tok/s) but fails on concurrent streams (both units locked to TP).
- **3x Spark (2+1)** meets main agent (75 tok/s) and has an independent sub-agent stream (50 tok/s), but maxes out at 2 concurrent streams.
- **RTX PRO 6000** meets all targets and handles 5-10 concurrent streams on a single card (at reduced per-stream speed).
- **Dense 70B** is a dead path on DGX Spark (~5 tok/s with TP=2). If dense models matter as a fallback, the PRO 6000 is the only option.

---

## Practical Considerations

### Software setup

Multi-node tensor parallelism requires more setup than a single-GPU inference server:

1. Connect units via QSFP cable
2. Configure network interfaces for NCCL communication
3. Set up passwordless SSH between nodes
4. Run vLLM or SGLang with tensor parallelism enabled across nodes
5. Use NVIDIA's Docker containers (recommended) or manual NCCL/MPI setup

NVIDIA provides official guides and Docker images. It works, but is more complex than `ollama run model` on a single GPU.

### ARM platform limitations

The GB10 uses ARM (Grace) CPUs, not x86. Most inference frameworks work (vLLM, SGLang, TRT-LLM, llama.cpp, Ollama), but:
- Some Python packages may not have ARM wheels
- Older CUDA libraries may need recompilation
- Community support and troubleshooting resources are smaller than x86+CUDA
- The ecosystem is improving rapidly (NVIDIA is investing heavily in DGX Spark tooling)

### Storage

- ASUS Ascent 1 TB: sufficient for 10-15 large models in Q4/FP8. Models can be downloaded on demand.
- NVIDIA FE 4 TB: comfortable for storing many models locally. Less important than it sounds — model files can be managed.
- External NVMe over USB4 is supported if more storage is needed.

### Noise and power

- Each unit is **laptop-class quiet** at 140W. Three units at 420W total is still quieter than a single RTX 5090 (575W with gaming fans).
- Standard Swiss outlets (10A/230V = 2,300W) have no issues. Even 4 units at 560W is well within limits.
- No PSU sizing, no thermal management, no airflow planning. Each unit is self-contained.

### Warranty

- ASUS Ascent: standard ASUS warranty (2 years CH). Available from major Swiss retailers (Brack, Galaxus, Foletti).
- NVIDIA FE: NVIDIA warranty. Available from Swiss shops via toppreise.ch.

---

## Upgrade Path

| Current | Upgrade | Cost | Result |
|---------|---------|------|--------|
| 2x Spark | Add 3rd unit | ~CHF 2,477 (ASUS) | 2+1 deployment: 75 tok/s main + 50 tok/s sub |
| 3x Spark | Add 4th unit | ~CHF 2,477 (ASUS) | 2+2 deployment: two independent TP=2 clusters at 75 tok/s each |
| 2x Spark | Add switch + 3rd unit | ~CHF 2,677+ | 3-node TP: ~85-95 tok/s estimated |

Each additional unit adds 128 GB memory. The incremental cost (~CHF 2,477) is far lower than adding a second RTX PRO 6000 (~CHF 7,691).

However, speed scaling is sublinear: each additional unit in a TP cluster adds ~40-50% of a single unit's speed, not 100%, due to network synchronization overhead.

---

## Recommendation

### DGX Spark is best if:
- **Budget is primary concern**: 2x ASUS Ascent at CHF 5,004 meets the 60-80 tok/s target at half the cost of the PRO 6000 build
- **Silence is paramount**: near-silent operation, no build required
- **You need massive memory**: 256-384 GB enables models that cannot run on 96 GB
- **You exclusively use MoE models**: Spark is fast enough for MoE (50-75 tok/s) but unusable for dense 70B

### RTX PRO 6000 is best if:
- **Speed matters most**: 134 tok/s vs 75 tok/s — 1.8x faster per-stream
- **Dense model fallback**: 32 tok/s on 70B vs ~5 tok/s
- **High concurrency**: single GPU handles 5-10 streams (shared bandwidth)
- **Software ecosystem**: x86 + CUDA is the broadest, most mature stack
- **Upgrade path**: NVLink to 192 GB at 3,584 GB/s combined bandwidth

### Hybrid option: PRO 6000 + 1x DGX Spark (~CHF 11,962)

Combine the best of both:
- RTX PRO 6000 AM5 build (CHF 9,485) → 134 tok/s main agent
- 1x ASUS Ascent (CHF 2,477) → dedicated 128 GB sub-agent / vision / large model overflow
- Total: ~CHF 11,962 — within the CHF 15,000 budget
- Best of both: speed from PRO 6000, memory depth from Spark

---

## Shop Links (Switzerland)

All links verified February 17, 2026.

### DGX Spark / ASUS Ascent Units

| Product | Storage | Price (CHF) | Link |
|---------|---------|-------------|------|
| **ASUS Ascent GX10-GG0003BN** (cheapest) | 1 TB | **2,477** | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) |
| ASUS Ascent GX10-GG0026BN | 2 TB | 2,908 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000U0-p824150) |
| ASUS Ascent GX10-GG0026BN | 4 TB | 3,285 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0026BN-NVIDIA-Grace-Blackwell-90MS0371-M000V0-p824154) |
| NVIDIA DGX Spark Founders Edition | 4 TB | 3,503 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/NVIDIA-DGX-Spark-Founders-Edition-Nvidia-GB10-10x-p825126) |
| PNY DGX Spark Founders Edition | 4 TB | 3,741 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Complete-systems/PNY-Nvidia-DGX-Spark-Founders-Edition-Nvidia-GB10-p823921) |

> **Buy the ASUS Ascent 1 TB** for cluster builds. Same GB10 chip, same 128 GB memory, same performance. The storage difference (1 TB vs 4 TB) is irrelevant — models are typically 20-65 GB each, and 1 TB holds 15-30 models. DGX OS can be installed on any GB10 unit.

---

## Sources

- [NVIDIA DGX Spark Product Page](https://www.nvidia.com/en-us/products/workstations/dgx-spark/)
- [NVIDIA DGX Spark Performance Blog](https://developer.nvidia.com/blog/how-nvidia-dgx-sparks-performance-enables-intensive-ai-tasks/)
- [LMSYS DGX Spark In-Depth Review](https://lmsys.org/blog/2025-10-13-nvidia-dgx-spark/)
- [LMSYS: Optimizing GPT-OSS on DGX Spark](https://lmsys.org/blog/2025-11-03-gpt-oss-on-nvidia-dgx-spark/)
- [DGX Spark Clustering Guide (NVIDIA Docs)](https://docs.nvidia.com/dgx/dgx-spark/spark-clustering.html)
- [Dual DGX Spark Setup & Benchmarks (NVIDIA Forum)](https://forums.developer.nvidia.com/t/setting-up-vllm-sglang-or-tensorrt-on-two-dgx-sparks/353338)
- [DGX Spark vs RTX 5090 Benchmarks (ProxPC)](https://www.proxpc.com/blogs/nvidia-dgx-spark-gb10-performance-test-vs-5090-llm-image-and-video-generation)
- [DGX Spark Alternatives & Benchmarks (AIMultiple)](https://research.aimultiple.com/dgx-spark-alternatives/)
