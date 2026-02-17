# NVIDIA GPU Research for Local AI Inference

> Research date: February 2026 | Budget: CHF 4,000–15,000

---

## GPU Comparison Summary

| GPU | VRAM | Memory BW | TDP | NVLink | Price (CHF) | MoE tok/s | Dense 70B tok/s |
|-----|------|-----------|-----|--------|-------------|-----------|-----------------|
| **RTX PRO 6000 Blackwell** | 96 GB GDDR7 ECC | 1,792 GB/s | 300W | Yes | [~7,691](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | **134** | ~32 |
| **RTX 5090** | 32 GB GDDR7 | 1,792 GB/s | 575W | No | [~2,224–3,500](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250) | — | ~27 (2x GPU) |
| RTX 4090 | 24 GB GDDR6X | 1,008 GB/s | 450W | No | [~1,700–2,900](https://www.toppreise.ch/productcollection/GeForce_RTX_4090-pc-s63606) | — | ~15-20 (2x GPU) |
| RTX 3090 (used) | 24 GB GDDR6X | 936 GB/s | 350W | 2-way | ~550-650 | — | ~10-15 (2x GPU) |
| RTX A6000 | 48 GB GDDR6 ECC | 768 GB/s | 300W | 2-way | [~4,046](https://www.toppreise.ch/price-comparison/Graphics-cards/ASUS-Nvidia-RTX-A6000-Nvidia-RTX-A6000-48GB-GDDR6-90SKC000-M5EAN0-p716143) (new) / ~1,900-3,300 (used) | — | ~12-18 (2x GPU) |
| L40S | 48 GB GDDR6 | 864 GB/s | 350W | No | ~7,500-10,000 | — | ~20-25 (2x GPU) |
| A100 80GB (used) | 80 GB HBM2e | 2,039 GB/s | 300W | Yes | ~5,100-7,000 | — | ~22-25 (single) |
| H100 PCIe | 80 GB HBM3 | 3,350 GB/s | 350W | Yes | ~22,000-30,000 | — | ~25-30 (single) |

### Key Insight: Memory Bandwidth Is Everything for Inference

Token generation is memory-bandwidth-bound. Each token requires reading model weights from memory:

**Dense model: tok/s ≈ Memory Bandwidth (GB/s) / Model Size in VRAM (GB)**
**MoE model: tok/s ≈ Memory Bandwidth (GB/s) / Active Parameters Size in VRAM (GB)**

For a 70B dense model in Q4 (~36 GB):
- RTX PRO 6000 / RTX 5090 (1,792 GB/s): ~50 tok/s theoretical → ~32 tok/s real (64% efficiency)
- A100 80GB (2,039 GB/s): ~57 tok/s theoretical → ~22-25 tok/s real

For GPT-OSS-120B MoE (~20B active, ~10 GB active weights Q4):
- RTX PRO 6000 (1,792 GB/s): ~179 tok/s theoretical → **134 tok/s** real (75% efficiency)
- Mac M3 Ultra (819 GB/s): ~82 tok/s theoretical → ~69 tok/s real

MoE models are the practical path to meeting 60-80 tok/s targets on prosumer hardware.

---

## RTX PRO 6000 Blackwell (96 GB) — RECOMMENDED

**Architecture**: Blackwell (GB202, full die)
**CUDA Cores**: 24,064 | **Tensor Cores**: 752
**VRAM**: 96 GB GDDR7 ECC, 512-bit bus
**Memory Bandwidth**: 1,792 GB/s
**TDP**: 300W | **Interface**: PCIe 5.0 x16 | **NVLink**: Yes
**FP4 Support**: Native Blackwell hardware (1.32x throughput over FP8)

**Swiss pricing (Feb 2026):**
- NVIDIA OEM Workstation Edition: from [CHF 7,691](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823)
- PNY Workstation Edition: from [CHF 7,848](https://www.toppreise.ch/price-comparison/Graphics-cards/PNY-Nvidia-RTX-Pro-6000-Blackwell-WS-Nvidia-VCNRTXPRO6000-PB-p804716)

**Inference benchmarks (Ollama, single card):**

| Model | Type | Params (total/active) | Quant | tok/s |
|---|---|---|---|---|
| GPT-OSS-120B | MoE | 120B / ~20B | Q4 | **134** |
| GPT-OSS-20B | MoE | 20B / ~5B | Q4 | **185** |
| DeepSeek-R1 | Dense | 32B / 32B | Q4 | 64 |
| Qwen 3 | Dense | 32B / 32B | Q4 | 56 |
| Gemma 3 | Dense | 27B / 27B | Q4 | 62 |
| LLaMA 3.3 | Dense | 70B / 70B | Q4 | 32 |
| Qwen 2.5 | Dense | 72B / 72B | Q4 | 29 |

**Why this GPU changes everything:**
- **96 GB on a single card** — fits MoE main agent (65 GB) + sub-agent (18 GB) + vision (5 GB) = 88 GB total
- **MoE at 134 tok/s** — exceeds the 60-80 tok/s target by 2x on a single card
- **Dense 70B at ~32 tok/s** — meets the 30 tok/s minimum as a fallback
- **300W TDP** — manageable power and noise for home office
- **NVLink support** — can pair two PRO 6000s for 192 GB if needed later
- **FP4 native support** — future MoE models in NVFP4 format run even faster

**Multi-card:** 2x on TRX50 (NVLink supported). 3x+ requires WRX90.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [NVIDIA RTX PRO 6000 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/data-center/rtx-pro-6000-blackwell-workstation-edition/workstation-blackwell-rtx-pro-6000-workstation-edition-nvidia-us-3519208-web.pdf), [StorageReview](https://www.storagereview.com/review/nvidia-rtx-pro-6000-workstation-gpu-review-blackwell-architecture-and-96-gb-for-pro-workflows)

---

## RTX 5090 (32 GB GDDR7)

**Architecture**: Blackwell (GB202) | **VRAM**: 32 GB GDDR7, 512-bit | **Bandwidth**: 1,792 GB/s
**TDP**: 575W | **NVLink**: Not supported | **Multi-GPU**: PCIe only

**Swiss pricing (Feb 2026):** [CHF 2,224–3,500](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250) depending on model. Limited stock.

**Inference benchmarks:**
- 8B model (single GPU): ~150-180 tok/s
- 70B Q4 (2x RTX 5090): ~27 tok/s (Ollama)
- 70B Q4 (2x RTX 5090, vLLM TP): likely 30-40 tok/s
- 67% improvement over RTX 4090 in LLM inference

**Noise:** 575W TDP = significant heat. Expect 40-50 dBA under sustained load. Two cards: 45-55 dBA.

**Physical limits:** 2x in a tower (practical maximum). 3-4x infeasible (575W x 4 = 2,300W GPU alone).

**Verdict:** Best single-card consumer performance. Two cards give 64 GB VRAM at ~CHF 4,800-7,000 — but 64 GB is well below the 96 GB practical minimum for multi-agent workloads. Best used as a companion to the PRO 6000 (Config #2 in [build-guides.md](build-guides.md)).

Sources: [RTX 5090 Europe Pricing](https://videocardz.com/newz/nvidia-geforce-rtx-5090-sees-first-price-drop-below-msrp-in-europe), [Dual RTX 5090 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2), [Dual RTX 5090 vs H100](https://www.hardware-corner.net/dual-rtx-5090-vs-h100-for-llm/)

---

## Other GPUs Considered

### RTX 4090 (24 GB GDDR6X) — Production ceased

Ada Lovelace (AD102). 1,008 GB/s bandwidth. 450W TDP. No NVLink. New: ~[CHF 1,700–2,900](https://www.toppreise.ch/productcollection/GeForce_RTX_4090-pc-s63606) (scarce, production stopped Oct 2024). Used: ~CHF 2,000-2,200. 70B Q4 on 2x: ~15-20 tok/s. Superseded by RTX 5090 (67% faster) and PRO 6000 (4x the VRAM at similar bandwidth).

Sources: [RTX 4090 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx4090), [RTX 4090 Pricing History](https://bestvaluegpu.com/en-eu/history/new-and-used-rtx-4090-price-history-and-specs/)

### RTX 3090 (24 GB GDDR6X) — Used only

Ampere (GA102). 936 GB/s bandwidth. 350W TDP. **NVLink supported (2-way)**. Used: ~CHF 550-650. Best VRAM-per-dollar: 4x = 96 GB for ~CHF 2,200-2,600. But 70B on 4x 3090: only ~15-20 tok/s — too slow for primary use.

Sources: [RTX 3090 Value for AI](https://www.xda-developers.com/used-rtx-3090-value-king-local-ai/), [RTX 3090 NVLink Review](https://www.servethehome.com/dual-nvidia-geforce-rtx-3090-nvlink-performance-review-asus-zotac/)

### RTX A6000 (48 GB GDDR6) — Professional, used market

Ampere (GA102 full die). 768 GB/s bandwidth (lowest in this comparison). 300W TDP. NVLink (2-way). New: ~[CHF 4,046](https://www.toppreise.ch/price-comparison/Graphics-cards/ASUS-Nvidia-RTX-A6000-Nvidia-RTX-A6000-48GB-GDDR6-90SKC000-M5EAN0-p716143). Used: ~CHF 1,900-3,300. 4x = 192 GB for ~CHF 7,600-13,200 — excellent VRAM density. But only ~20-30 tok/s on 70B and blower-style coolers are loud. Superseded by PRO 6000 for inference.

Sources: [RTX A6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-a6000), [4x A6000 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-a6000-4)

### L40S (48 GB GDDR6) — Passive cooling dealbreaker

Ada Lovelace (AD102 datacenter). 864 GB/s bandwidth. 350W TDP. **Passive cooling only** — requires server chassis with forced airflow. Tower deployment risky without custom cooling. CHF 7,000-9,300 per card — 2x already blows budget. Not practical for home office.

Source: [L40S vs A100](https://acecloud.ai/blog/nvidia-l40s-vs-a100-ai-inference/)

### A100 80GB (HBM2e) — Best bandwidth, passive cooling

Ampere (GA100). 2,039 GB/s bandwidth (excellent). 300W TDP. **Passive cooling** — same tower issue as L40S. Used: ~CHF 5,100-7,000. Single A100 80GB: ~22-25 tok/s on 70B. 2x = 160 GB VRAM but CHF 10,200-14,000 in GPUs alone, leaving little room for the system. Prices expected to drop 30-45% through 2026.

Sources: [A100 Pricing Forecast](https://levelupblogs.com/news/nvidia-a100-80gb-gpu-price-prediction-2026/), [A100 40GB vs 80GB](https://verda.com/blog/nvidia-a100-40gb-vs-80-gb)

### H100 PCIe (80 GB HBM3) — Over budget

Hopper (GH100). 3,350 GB/s bandwidth (best). A single used H100 PCIe costs CHF 11,200-16,800 — at or above the entire budget. Definitively out of scope.

Source: [H100 Pricing Guide](https://docs.jarvislabs.ai/blog/h100-price)

---

## Multi-GPU Considerations

### Interconnect Bandwidth

| Interconnect | Bandwidth | Scaling (2 GPUs) |
|---|---|---|
| PCIe 4.0 x16 | ~32 GB/s per direction | ~1.5-1.7x |
| PCIe 5.0 x16 | ~64 GB/s per direction | ~1.6-1.8x |
| NVLink (PRO 6000/A6000/3090) | ~112.5 GB/s bidirectional | ~1.7-1.9x |

During tensor parallelism, GPUs synchronize after each transformer layer (all-reduce operations). On PCIe, this adds 20-40% overhead vs NVLink. Consumer GPUs (RTX 5090, 4090) without NVLink see diminishing returns beyond 2 GPUs.

### Tensor Parallelism vs Independent Models

- **Tensor parallelism (TP):** Splits layers across GPUs. Better latency for single requests. Needs fast interconnect.
- **Independent models on separate GPUs:** No interconnect needed. Best for multi-agent workflows where each GPU serves different models.

For multi-agent inference: prefer independent models on separate GPUs over tensor parallelism when possible.

### Platform Options

| Scenario | Platform | Why |
|---|---|---|
| 1 GPU | AM5 (Ryzen 9 9950X) | Full x16 bandwidth. Cheapest platform (~CHF 628). |
| 2 GPUs, independent models | AM5 X870E | PCIe x8/x8 is fine — GPUs don't communicate. |
| 2 GPUs, split model or NVLink | TRX50 (Threadripper 7960X) | Full x16/x16 + NVLink support (~CHF 1,608). |
| 3-4 GPUs | WRX90 (Threadripper PRO) | 128 PCIe 5.0 lanes. Required for 4+ GPU builds. |

### Inference Software

| Engine | Multi-GPU TP | Continuous Batching | Best For |
|---|---|---|---|
| **vLLM** | Yes | Yes | Production multi-GPU serving |
| **SGLang** | Yes | Yes | Same as vLLM, more stable under load |
| **ExLlamaV2/V3** | Limited | Limited | Consumer GPUs, single-card speed |
| **llama.cpp** | Layer split | Limited | Single-user low latency |
| **Ollama** | No | Limited | Quick setup, development |

Sources: [Multi-GPU TP Guide](https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/), [llama.cpp Multi-GPU Breakthrough](https://medium.com/@jagusztinl/llama-cpp-performance-breakthrough-for-multi-gpu-setups-04c83a66feb2), [NVLink vs PCIe](https://www.hyperstack.cloud/blog/case-study/nvlink-vs-pcie-whats-the-difference-for-ai-workloads)

---

## Sources

- [RTX 5090 Europe Pricing](https://videocardz.com/newz/nvidia-geforce-rtx-5090-sees-first-price-drop-below-msrp-in-europe)
- [RTX 5090 Toppreise.ch](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250)
- [RTX 5090 LLM Benchmarks (RunPod)](https://www.runpod.io/blog/rtx-5090-llm-benchmarks)
- [RTX 5090 Bandwidth Impact on LLMs](https://blog.neevcloud.com/the-impact-of-rtx-5090s-memory-bandwidth-on-llms)
- [Dual RTX 5090 vs H100](https://www.hardware-corner.net/dual-rtx-5090-vs-h100-for-llm/)
- [Dual RTX 5090 Benchmark (DatabaseMart)](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2)
- [RTX 5090 Specs (GamersNexus)](https://gamersnexus.net/gpus/nvidia-geforce-rtx-5090-founders-edition-review-benchmarks-gaming-thermals-power)
- [RTX 4090 Benchmark (DatabaseMart)](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx4090)
- [RTX 4090 Pricing History](https://bestvaluegpu.com/en-eu/history/new-and-used-rtx-4090-price-history-and-specs/)
- [RTX 3090 Value for AI](https://www.xda-developers.com/used-rtx-3090-value-king-local-ai/)
- [RTX 3090 NVLink Review](https://www.servethehome.com/dual-nvidia-geforce-rtx-3090-nvlink-performance-review-asus-zotac/)
- [RTX A6000 Benchmark (DatabaseMart)](https://www.databasemart.com/blog/ollama-gpu-benchmark-a6000)
- [4x A6000 vLLM Benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-a6000-4)
- [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000)
- [NVIDIA RTX PRO 6000 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/data-center/rtx-pro-6000-blackwell-workstation-edition/workstation-blackwell-rtx-pro-6000-workstation-edition-nvidia-us-3519208-web.pdf)
- [StorageReview PRO 6000](https://www.storagereview.com/review/nvidia-rtx-pro-6000-workstation-gpu-review-blackwell-architecture-and-96-gb-for-pro-workflows)
- [A100 Pricing Forecast](https://levelupblogs.com/news/nvidia-a100-80gb-gpu-price-prediction-2026/)
- [L40S vs A100](https://acecloud.ai/blog/nvidia-l40s-vs-a100-ai-inference/)
- [H100 Pricing Guide](https://docs.jarvislabs.ai/blog/h100-price)
- [GPU Ranking for LLMs](https://www.hardware-corner.net/gpu-ranking-local-llm/)
- [Best GPUs for LLM Inference](https://localllm.in/blog/best-gpus-llm-inference-2025)
- [vLLM Parallelism and Scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)
- [Multi-GPU TP Guide](https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/)
- [llama.cpp Multi-GPU Breakthrough](https://medium.com/@jagusztinl/llama-cpp-performance-breakthrough-for-multi-gpu-setups-04c83a66feb2)
- [NVLink vs PCIe](https://www.hyperstack.cloud/blog/case-study/nvlink-vs-pcie-whats-the-difference-for-ai-workloads)
- [PCIe Considerations for Multi-GPU](https://aightbits.com/2025/04/15/diy-ai-pcie-considerations-for-multi-gpu-builds/)
