# NVIDIA GPU Research for Local AI Inference Build

**Date**: February 2026
**Budget**: 4,000 - 15,000 CHF (complete system)
**Target**: 96-256 GB VRAM, MoE coding models @ 60-80 tok/s + 5-10 concurrent sub-agents @ 15-20 tok/s + concurrent image recognition

---

## Table of Contents

1. [GPU Comparison Summary](#1-gpu-comparison-summary)
2. [Consumer GPUs: Detailed Analysis](#2-consumer-gpus-detailed-analysis)
3. [Professional/Enterprise GPUs: Detailed Analysis](#3-professionalenterprise-gpus-detailed-analysis)
4. [Multi-GPU System Considerations](#4-multi-gpu-system-considerations)
5. [Build Configurations Within Budget](#5-build-configurations-within-budget)
6. [Recommendations](#6-recommendations)

---

## 1. GPU Comparison Summary

### Quick Reference Table

| GPU | VRAM | Memory BW | TDP | NVLink? | New Price (CHF) | Used Price (CHF) | 70B tok/s (est.) |
|-----|------|-----------|-----|---------|-----------------|-------------------|------------------|
| **RTX 5090** | 32 GB GDDR7 | 1,792 GB/s | 575W | No | ~2,400-3,500 | N/A | ~27 t/s (2x GPU) |
| **RTX PRO 6000 Blackwell** | 96 GB GDDR7 ECC | 1,792 GB/s | 300W | **Yes (NVLink)** | ~6,443 | N/A | ~134 t/s MoE / ~32 t/s dense |
| **RTX 4090** | 24 GB GDDR6X | 1,008 GB/s | 450W | No | ~2,800 | ~2,000-2,200 | ~15-20 t/s (2x GPU, Q4) |
| **RTX 3090** | 24 GB GDDR6X | 936 GB/s | 350W | **Yes (2-way)** | Discontinued | ~550-650 | ~10-15 t/s (2x GPU, Q4) |
| **RTX A6000** | 48 GB GDDR6 ECC | 768 GB/s | 300W | **Yes (2-way)** | ~4,600 | ~2,000-3,000* | ~12-18 t/s (2x GPU) |
| **L40S** | 48 GB GDDR6 | 864 GB/s | 350W | No | ~7,500-10,000 | N/A | ~20-25 t/s (2x GPU) |
| **A100 80GB PCIe** | 80 GB HBM2e | 2,039 GB/s | 300W | Yes (NVSwitch) | EOL | ~5,500-7,500 | ~22-25 t/s (single) |
| **A100 40GB PCIe** | 40 GB HBM2e | 1,555 GB/s | 250W | Yes (NVSwitch) | EOL | ~3,500-5,000 | ~18-20 t/s (single, Q4) |
| **H100 PCIe** | 80 GB HBM3 | 3,350 GB/s | 350W | Yes | ~22,000-30,000 | ~12,000-18,000 | ~25-30 t/s (single) |

*A6000 used prices are highly variable; expect EUR 2,000-3,500 depending on source and condition.*

### Key Insight: Memory Bandwidth Is Everything for Inference

For LLM token generation (the bottleneck use case), performance scales almost linearly with memory bandwidth. This is because during autoregressive generation, every token requires reading the entire model weights from memory once. The formula is roughly:

**Token generation speed (tok/s) = Memory Bandwidth (GB/s) / Model Size in Memory (GB)**

For a 70B model in Q4 quantization (~35-40 GB):
- RTX 5090 (1,792 GB/s): ~45-51 tok/s theoretical, ~27 tok/s real on 2 GPUs
- RTX 4090 (1,008 GB/s): ~25-29 tok/s theoretical per GPU
- A100 80GB (2,039 GB/s): ~51-58 tok/s theoretical, ~22-25 tok/s real
- H100 (3,350 GB/s): ~84-96 tok/s theoretical, ~25-30 tok/s real

Real-world numbers are lower due to overhead, but the ranking holds.

**MoE Model Update (February 2026):** The above formula applies to dense models where all parameters are read per token. MoE (Mixture of Experts) models only activate a subset of parameters (e.g., 20B out of 120B total), dramatically changing the equation:

**MoE tok/s ≈ Memory Bandwidth / Active Parameters in Memory**

For GPT-OSS-120B (~20B active, ~10 GB active weights in Q4):
- RTX PRO 6000 (1,792 GB/s): ~134 tok/s real — **far exceeds 60-80 target**
- Mac M3 Ultra (819 GB/s): ~69 tok/s real — **meets target**

This is why MoE models are the practical path to meeting speed targets on prosumer hardware.

---

## 2. Consumer GPUs: Detailed Analysis

### 2.1 RTX 5090 (32 GB GDDR7)

**Architecture**: Blackwell (GB202)
**CUDA Cores**: 21,760
**VRAM**: 32 GB GDDR7, 512-bit bus
**Memory Bandwidth**: 1,792 GB/s (77% more than RTX 4090)
**TDP**: 575W
**Interface**: PCIe 5.0 x16
**NVLink**: Not supported
**Multi-GPU**: PCIe only (tensor parallelism via software)

**Pricing in Switzerland (Feb 2026)**:
- Digitec/Galaxus: CHF 2,400 - 3,500 depending on model
- Toppreise.ch lowest: ~CHF 2,339 (MSI Ventus 3X OC)
- Premium models (ASUS ROG Astral, MSI Suprim): CHF 3,000-3,500+
- Availability: Limited stock, sells out quickly

**Inference Benchmarks**:
- 8B model (single GPU): ~150-180 tok/s (Ollama/llama.cpp)
- 7B model batch (vLLM, batch=8): ~5,841 tok/s total throughput
- 70B model Q4 (2x RTX 5090): ~27 tok/s generation (Ollama benchmark)
- 70B model (2x RTX 5090, vLLM tensor parallel): likely 30-40 tok/s
- Matches or exceeds single H100 performance on 70B at 2x GPU config
- 67% improvement over RTX 4090 in LLM inference

**Noise Considerations**:
- 575W TDP means significant heat dissipation needed
- Dual-fan and triple-fan AIB cards available
- Under sustained inference load, expect 40-50 dBA per card
- Two cards in a tower: 45-55 dBA -- audible but manageable with good case
- Water-cooled variants available (ASUS ROG Astral LC, MSI Suprim Liquid)

**Can You Fit 2, 3, or 4 in a System?**
- 2x: Yes, on AM5 X870E (x8/x8 PCIe 5.0) or Threadripper (x16/x16)
- 3x or 4x: Theoretically possible on Threadripper WRX90 but extreme challenges:
  - Physical space (most cards are 3-3.5 slot designs)
  - 575W x 4 = 2,300W GPU power alone -- not feasible in a standard build
  - Would need 240V circuit and 2x PSUs
- **Practical maximum: 2x in a tower**

**Verdict**: Best single-card performance for inference. Two cards give 64 GB VRAM. At ~CHF 4,800-7,000 for 2 cards, leaves room for the rest of the system at higher budgets. But 64 GB is well below the 128 GB minimum VRAM target.

---

### 2.2 RTX 4090 (24 GB GDDR6X)

**Architecture**: Ada Lovelace (AD102)
**CUDA Cores**: 16,384
**VRAM**: 24 GB GDDR6X, 384-bit bus
**Memory Bandwidth**: 1,008 GB/s
**TDP**: 450W
**Interface**: PCIe 4.0 x16
**NVLink**: Not supported (removed from 40-series consumer)
**Multi-GPU**: PCIe only

**Pricing (Feb 2026)**:
- New: ~CHF 2,800 (limited availability, production stopped Oct 2024)
- Used (Europe): ~EUR 2,000-2,200 (~CHF 2,000-2,200)
- Prices have NOT dropped significantly due to production halt and high demand

**Inference Benchmarks**:
- 8B model (single GPU): ~112-128 tok/s
- 70B model Q4 (single GPU, CPU offload): ~8-12 tok/s
- 70B model Q4 (2x GPU, tensor parallel): ~15-20 tok/s
- 70B model Q3 (fits in 24GB x2 = 48GB): ~12-15 tok/s
- 7B model: ~80-100 tok/s

**Noise**: Similar to 5090 concerns. 450W per card, triple-fan designs.

**Can You Fit Multiple?**
- 2x: Easy on consumer/HEDT platforms
- 3x: Possible on Threadripper
- 4x: Requires Threadripper PRO WRX90, significant cooling, ~1,800W GPU power

**Verdict**: Still powerful but 24 GB VRAM per card is limiting. 4x gives only 96 GB (below 128 GB minimum). No NVLink means memory can't be pooled. With 2x at used prices (~CHF 4,000-4,400), you get only 48 GB VRAM. Poor value proposition vs. RTX 5090 or used professional cards at this point.

---

### 2.3 RTX 3090 (24 GB GDDR6X)

**Architecture**: Ampere (GA102)
**CUDA Cores**: 10,496
**VRAM**: 24 GB GDDR6X, 384-bit bus
**Memory Bandwidth**: 936 GB/s
**TDP**: 350W
**Interface**: PCIe 4.0 x16
**NVLink**: Yes! 2-way NVLink supported (112.5 GB/s bidirectional)
**Multi-GPU**: NVLink (2 GPUs) + PCIe (additional GPUs)

**Pricing (Feb 2026)**:
- Used (eBay Europe): ~EUR 550-650 (~CHF 550-650)
- Extremely good value per GB of VRAM at this price point
- Widely available on the secondary market

**Inference Benchmarks**:
- 8B model (single GPU): ~100-112 tok/s
- 7B model (Llama2-7B): ~45 tok/s
- 13B model: ~20-25 tok/s
- 70B Q4 (4x GPU): ~15-20 tok/s (depends heavily on PCIe topology)
- 70B Q4 (2x NVLinked): ~10-15 tok/s

**NVLink Advantage**: 2x RTX 3090 with NVLink effectively creates a 48 GB unified memory pool at 112.5 GB/s link speed. This is meaningful for models that exceed single-GPU VRAM but the link speed is still far below each GPU's internal 936 GB/s bandwidth.

**Can You Fit Multiple?**
- 2x NVLink: Yes, common and well-supported
- 4x: 2 pairs with NVLink bridges possible on Threadripper; the other 2 GPUs communicate via PCIe
- Power: 4x = 1,400W GPU power -- needs 1,600W+ PSU, possible on 230V
- Physical: 3090 FE is a 3-slot card; most AIBs are 3-3.5 slots

**Verdict**: The value king for VRAM-per-dollar. 4x RTX 3090 = 96 GB VRAM for ~CHF 2,200-2,600 in GPUs. With a Threadripper system, total build ~CHF 5,000-7,000. However: 96 GB is below the 128 GB minimum, bandwidth is lower than newer cards, and 70B at 60+ tok/s is NOT achievable. Best case for 70B on 4x 3090 is ~15-20 tok/s. Falls short of performance targets significantly.

---

## 3. Professional/Enterprise GPUs: Detailed Analysis

### 3.1 RTX A6000 (48 GB GDDR6)

**Architecture**: Ampere (GA102, full die)
**CUDA Cores**: 10,752
**VRAM**: 48 GB GDDR6 ECC, 384-bit bus
**Memory Bandwidth**: 768 GB/s
**TDP**: 300W
**Interface**: PCIe 4.0 x16
**NVLink**: Yes! 2-way NVLink (112.5 GB/s bidirectional)
**Cooling**: Blower-style (designed for multi-GPU workstations)
**Form Factor**: Dual-slot, full-height

**Pricing (Feb 2026)**:
- New: ~USD 4,600 (~CHF 4,300)
- Used (eBay/secondary): ~USD 2,000-3,500 (~CHF 1,900-3,300) -- highly variable
- Refurbished from specialized dealers: ~USD 2,500-3,500
- Risk: Used enterprise cards may have been run 24/7; verify thermal history

**Inference Benchmarks**:
- 8B model (single GPU): ~100 tok/s
- 7B model (vLLM, concurrent): ~1,600 tok/s total throughput
- 70B Q4 (2x NVLink, 96 GB pool): ~15-20 tok/s
- 70B Q4 (4x A6000, vLLM tensor parallel): ~420-470 tok/s batch throughput, ~18-25 tok/s per-request

**NVLink Configuration**:
- 2x A6000 + NVLink bridge = 96 GB unified VRAM at 112.5 GB/s
- 4x A6000 = 2 NVLink pairs; each pair has 96 GB unified; across pairs communication falls to PCIe
- **Important**: NVLink bridges for A6000 are compatible with RTX 3090 bridges (same connector)

**Noise Considerations**:
- Blower-style cooler: louder than open-air consumer coolers at load
- Designed for multi-GPU stacking: maintains consistent temps in dense configs
- At 300W TDP, quieter than 5090 (575W) or 4090 (450W)
- 4x A6000 at full load: expect 50-60 dBA -- noticeable in a home office
- Aftermarket cooling (e.g., Noctua duct mod or water cooling) can reduce noise significantly

**Can You Fit Multiple?**
- 2x: Easy on any platform with 2 PCIe x16 slots
- 4x: Possible on Threadripper PRO WRX90; dual-slot design fits well
- Power: 4x = 1,200W GPU power -- very manageable with 1,600W PSU

**Verdict**: The most interesting option for hitting 128+ GB VRAM within budget. 4x used A6000 = 192 GB VRAM for ~CHF 7,600-13,200 in GPUs. However, the 768 GB/s bandwidth per card is the lowest in this comparison -- this limits per-request token generation speed. With 4x cards and tensor parallelism, the combined bandwidth helps, but PCIe communication overhead between the 2 NVLink pairs is significant. Probably achieves 20-30 tok/s on 70B, falling short of the 60-80 tok/s target.

---

### 3.2 NVIDIA L40S (48 GB GDDR6)

**Architecture**: Ada Lovelace (AD102, data center variant)
**CUDA Cores**: 18,176
**VRAM**: 48 GB GDDR6, 384-bit bus
**Memory Bandwidth**: 864 GB/s
**TDP**: 350W
**Interface**: PCIe 4.0 x16
**NVLink**: Not supported
**Cooling**: Passive only -- no onboard fan

**Pricing (Feb 2026)**:
- New: ~USD 7,500-10,000 (~CHF 7,000-9,300)
- Used: Very limited secondary market
- Total system cost for 4x: would exceed budget significantly

**Inference Benchmarks**:
- 8B model (batch=1): ~44 tok/s
- 8B model (batch=8): ~325 tok/s
- 70B Q4: ~20-25 tok/s (2x GPU)
- Roughly 50-55% of H100 throughput
- Better than A6000 due to Ada architecture and FP8 support

**CRITICAL ISSUE: Passive Cooling**:
- The L40S has NO onboard fan -- it is designed for servers with front-to-back forced airflow
- In a tower/workstation, you MUST provide dedicated airflow (e.g., case fans directly blowing over the heatsink, or liquid cooling)
- Without proper airflow: thermal throttling and potential hardware damage
- This makes tower deployment risky and complex

**Verdict**: Better performance than A6000 per card, but passive cooling is a dealbreaker for home office use without custom cooling solutions. At CHF 7,000-9,300 per card, 2x already blows the budget with no room for the rest of the system. Not practical for this build.

---

### 3.3 NVIDIA A100 (40 GB / 80 GB)

**Architecture**: Ampere (GA100)
**VRAM**: 40 GB or 80 GB HBM2e
**Memory Bandwidth**: 1,555 GB/s (40 GB) / 2,039 GB/s (80 GB)
**TDP**: 250W (40GB PCIe) / 300W (80GB PCIe) / 400W (SXM)
**Interface**: PCIe 4.0 x16 (PCIe variant) or SXM4
**NVLink**: Yes (SXM4 version with NVSwitch); PCIe version limited
**Cooling**: PCIe version has passive heatsink (same issue as L40S)

**Pricing (Feb 2026)**:
- A100 40GB PCIe used: ~USD 3,500-5,000 (~CHF 3,300-4,700)
- A100 80GB PCIe used: ~USD 5,500-7,500 (~CHF 5,100-7,000)
- A100 80GB SXM4 used: ~USD 4,000-6,000 but requires specialized baseboard
- Production ceased February 2024 -- EOL product
- Prices predicted to drop 30-45% through 2026 as enterprises decommission for Blackwell

**Inference Benchmarks (80 GB)**:
- 8B model: ~80-100 tok/s
- 70B Q4 (single GPU, fits in 80GB): ~22-25 tok/s
- 70B FP16 (2x 80GB, 160GB total): ~20-22 tok/s
- HBM2e bandwidth advantage is very noticeable vs GDDR6 cards

**Inference Benchmarks (40 GB)**:
- 70B Q4 (requires 2x 40GB): ~18-20 tok/s
- Lower bandwidth than 80GB variant reduces throughput

**CRITICAL ISSUE: Passive Cooling (Same as L40S)**:
- PCIe A100 cards are passively cooled -- designed for server chassis with forced airflow
- Tower deployment requires aftermarket cooling or high-CFM case fans
- SXM4 variants require proprietary DGX/HGX baseboards -- not usable in a tower at all

**Verdict**: Excellent memory bandwidth (especially 80GB model) makes these strong inference cards. 2x A100 80GB = 160 GB VRAM with outstanding 2,039 GB/s bandwidth per card. At ~CHF 10,200-14,000 for 2x used, leaves limited room for the system. The passive cooling problem is real but solvable with aftermarket solutions. For 70B inference, a single A100 80GB achieves ~22-25 tok/s -- about half the minimum target. 2x would need tensor parallelism, which on PCIe adds overhead, likely yielding 30-40 tok/s. Still short of 60-80 tok/s target.

---

### 3.4 NVIDIA H100 (80 GB)

**Architecture**: Hopper (GH100)
**VRAM**: 80 GB HBM3
**Memory Bandwidth**: 3,350 GB/s (SXM5) / 2,000 GB/s (PCIe)
**TDP**: 350W (PCIe) / 700W (SXM5)
**Interface**: PCIe 5.0 x16 or SXM5
**NVLink**: Yes (4th gen, 900 GB/s on SXM5)

**Pricing (Feb 2026)**:
- H100 PCIe 80GB new: ~USD 22,000-30,000 (~CHF 20,500-28,000)
- H100 SXM5 80GB new: ~USD 35,000-40,000
- Used H100 PCIe: ~USD 12,000-18,000 (~CHF 11,200-16,800)
- Cloud rental has dropped to ~$1.50-2.80/hr

**Inference Benchmarks**:
- 70B Q4 (single GPU): ~25-30 tok/s
- 70B FP8 (single GPU): ~30-35 tok/s
- 8B model: ~150-200 tok/s

**Verdict**: Reality check -- a SINGLE used H100 PCIe card costs CHF 11,200-16,800, which is AT or ABOVE the entire budget. Even if you found one at the low end, you'd have 80 GB VRAM (below 128 GB minimum) and no money for the rest of the system. The H100 is definitively out of budget for this build. Cloud rental at $1.50-2.80/hr may be worth considering as a supplement.

---

### 3.5 RTX PRO 6000 Blackwell (96 GB GDDR7) — RECOMMENDED

**Architecture**: Blackwell (GB202, full die)
**CUDA Cores**: 24,064
**Tensor Cores**: 752
**VRAM**: 96 GB GDDR7 ECC, 512-bit bus
**Memory Bandwidth**: 1,792 GB/s
**TDP**: 300W
**Interface**: PCIe 5.0 x16
**NVLink**: Yes (professional feature)
**Cooling**: Active blower (Workstation Edition) / Passive (Server Edition)
**FP4 Support**: Yes (native Blackwell hardware, 1.32x throughput over FP8)

**Pricing in Switzerland (Feb 2026)**:
- Workstation Edition (NVIDIA): from CHF 6,443 ([Toppreise.ch](https://www.toppreise.ch))
- Server Edition (NVIDIA): from CHF 6,749
- PNY Workstation Edition: from CHF 7,848

**Inference Benchmarks (Ollama, single card):**

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

**Can You Fit Multiple?**
- 2x: Yes, on TRX50 or WRX90. NVLink supported.
- 3x+: Requires WRX90 (128 PCIe lanes)
- Power: 2x = 600W GPU + system = ~800W total — very manageable

**Verdict**: The clear winner for AI inference. A single card meets all performance targets with MoE models. 96 GB is sufficient for concurrent model loading. At CHF 6,443, leaves room in the budget for the rest of the system (~CHF 8,200 total). The upgrade path is straightforward: add a second PRO 6000 for 192 GB.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [NVIDIA RTX PRO 6000 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/data-center/rtx-pro-6000-blackwell-workstation-edition/workstation-blackwell-rtx-pro-6000-workstation-edition-nvidia-us-3519208-web.pdf), [StorageReview](https://www.storagereview.com/review/nvidia-rtx-pro-6000-workstation-gpu-review-blackwell-architecture-and-96-gb-for-pro-workflows), [Toppreise.ch](https://www.toppreise.ch)

---

## 4. Multi-GPU System Considerations

### 4.1 PCIe Bandwidth: The Elephant in the Room

For multi-GPU tensor parallelism without NVLink, GPUs communicate over PCIe:

| Interconnect | Bandwidth | Latency |
|---|---|---|
| PCIe 4.0 x16 | ~32 GB/s (each direction) | Higher |
| PCIe 5.0 x16 | ~64 GB/s (each direction) | Higher |
| NVLink (RTX 3090/A6000) | ~112.5 GB/s (bidirectional) | Lower |
| NVLink 4th Gen (H100 SXM5) | ~900 GB/s (bidirectional) | Lowest |
| GPU Internal Memory BW | 768-2,039 GB/s | -- |

**Impact on Inference**: During tensor parallelism, GPUs must synchronize after each transformer layer (all-reduce operations). On PCIe, this adds 20-40% overhead vs NVLink. The impact is:
- 2 GPUs over PCIe: expect ~1.5-1.7x speedup (not 2x)
- 4 GPUs over PCIe: expect ~2.5-3.5x speedup (not 4x)
- The bottleneck worsens with larger models and longer sequences

**Practical Implication**: Consumer GPUs (RTX 5090, 4090) without NVLink will see diminishing returns beyond 2 GPUs. RTX 3090 and A6000 with NVLink are better for 2-GPU pairs, but cross-pair communication still falls to PCIe.

**Software Matters**: The inference engine choice significantly affects multi-GPU performance:
- **vLLM**: Best for tensor parallelism and batch inference. Recommended for multi-GPU.
- **ExLlamaV2/V3**: Excellent for consumer GPUs. V3 achieves 120 tok/s on a single RTX 5090 for supported models.
- **llama.cpp**: Recent "Graph Split" mode achieves 3-4x improvement over old layer split mode. Now competitive for multi-GPU.
- **Ollama**: Wraps llama.cpp; simpler but inherits its multi-GPU limitations.

### 4.2 Motherboard / Platform Options

**For 2x GPUs (Consumer / HEDT)**:
- AMD AM5 X870E boards: PCIe 5.0 x8/x8 with 2 cards. ~CHF 300-500.
- Intel Z890/B860: Similar PCIe 5.0 support. ~CHF 300-500.
- These work fine for 2x consumer GPUs.

**For 3-4x GPUs (Workstation/HEDT)**:
- **AMD Threadripper 7000 (TRX50)**: 48 PCIe 5.0 lanes. Supports 3x GPUs at x16/x16/x16. CPU starts at ~CHF 1,500.
- **AMD Threadripper PRO 7000/9000 (WRX90)**: 128 PCIe 5.0 lanes. Supports 4x GPUs at full x16 bandwidth. CPU: CHF 1,600-11,700. Motherboards: CHF 800-1,500.
- **ASUS Pro WS WRX90E-SAGE**: Popular 4-GPU motherboard. ~CHF 1,000-1,200.

**Cost Impact of Threadripper**:
- Entry Threadripper PRO 9955WX: ~CHF 1,600
- WRX90 motherboard: ~CHF 1,000-1,200
- 128 GB DDR5 ECC RAM: ~CHF 400-600
- Total platform cost: ~CHF 3,000-3,500 (just CPU + mobo + RAM)
- This is significant for a 4-GPU build within budget

### 4.3 Power Supply Requirements

| Configuration | GPU Power | System Total | PSU Needed |
|---|---|---|---|
| 2x RTX 5090 | ~1,150W | ~1,400W | 1,600W |
| 2x RTX 4090 | ~900W | ~1,150W | 1,300W |
| 4x RTX 3090 | ~1,400W | ~1,650W | 1,800-2,000W |
| 4x RTX A6000 | ~1,200W | ~1,450W | 1,600W |
| 2x A100 80GB PCIe | ~600W | ~850W | 1,000W |

- Swiss residential power: 230V / 10A standard outlet = 2,300W max per circuit
- 16A circuit (common in Switzerland) = 3,680W max
- All configurations above fit within a single 16A circuit
- For 4x GPU builds, a high-quality 1,600W PSU (e.g., Corsair HX1600i, be quiet! Dark Power Pro 13 1600W) costs ~CHF 400-500

### 4.4 Cooling and Noise in a Tower

**The Challenge**: Multi-GPU builds generate 600-2,300W of heat in a confined space.

**Open-Air Consumer GPUs (5090, 4090, 3090)**:
- Pros: Good cooling performance, reasonable noise for 1-2 cards
- Cons: Exhaust heat into the case; 2nd GPU runs hotter (10-15C warmer)
- 2x cards: 45-55 dBA under sustained load
- 4x cards: 55-65+ dBA -- uncomfortable for a home office

**Blower-Style Professional GPUs (A6000)**:
- Pros: Exhaust heat out the back of the case; designed for stacking
- Cons: Blower fans are inherently louder at given RPM
- 2x cards: 50-55 dBA
- 4x cards: 55-65 dBA
- The A6000's 300W TDP helps vs. higher-TDP cards

**Passive Data Center GPUs (L40S, A100 PCIe)**:
- NO onboard cooling -- requires dedicated airflow solution
- Options: High-CFM case fans, aftermarket heatsink mods, liquid cooling
- Can actually be quieter IF you invest in a proper liquid cooling loop

**Recommended Quiet Cases for Multi-GPU**:
- **be quiet! Dark Base 900 Pro**: Full tower, noise dampened, good airflow
- **Fractal Design Define 7 XL**: Large, sound-dampened, fits E-ATX
- **Lian Li O11 Dynamic EVO XL**: Open layout, good for watercooling multi-GPU
- Use large (140mm) case fans at low RPM for ambient noise reduction

**Water Cooling**: For the quietest multi-GPU setup, a custom loop with GPU waterblocks is the best option. Cost: CHF 500-1,000 for a 2-GPU loop. Reduces noise to 30-40 dBA even under full load.

### 4.5 Tensor Parallelism vs. Pipeline Parallelism for Inference

**Tensor Parallelism (TP)**: Splits each layer across GPUs. All GPUs work simultaneously on every token. Better latency for single requests. Requires fast inter-GPU communication (NVLink ideal, PCIe workable).

**Pipeline Parallelism (PP)**: Each GPU handles different layers. One GPU waits while another processes. Lower single-request latency improvement but can be better for batched inference. Less communication overhead.

**For your use case (low-batch, latency-sensitive agent inference)**:
- Tensor Parallelism is preferred for the main 70B agent (single request, needs speed)
- Pipeline Parallelism or independent GPU assignment for sub-agents
- Mixed strategy: Main model on 2 GPUs with TP, sub-agents on remaining GPUs independently

---

## 5. Build Configurations Within Budget

### Configuration A: 4x RTX 3090 -- "Budget VRAM Machine"
**Total VRAM**: 96 GB (below 128 GB minimum)
**Estimated Cost**: CHF 5,500 - 7,500

| Component | Spec | Est. CHF |
|---|---|---|
| 4x RTX 3090 (used) | 4x 24GB = 96GB | 2,200-2,600 |
| CPU | AMD Threadripper PRO 9955WX | 1,600 |
| Motherboard | ASUS WRX90E-SAGE | 1,000 |
| RAM | 128GB DDR5 ECC | 400 |
| PSU | 1,800W Platinum | 450 |
| Storage | 2TB NVMe | 150 |
| Case | Full tower | 200 |
| NVLink bridges (2x) | For 2 pairs | 100-200 |
| **Total** | | **~6,100 - 6,600** |

**Performance Estimate**:
- 70B Q4 (4x GPU, tensor parallel): ~15-22 tok/s -- **FAILS 60-80 tok/s target, meets minimum 30 tok/s only with optimistic estimates**
- 8B sub-agents (1 GPU each): ~100 tok/s -- exceeds target
- Concurrent streams: Can run 2 sub-agents on 2 GPUs + 70B on 2 GPUs -- limited to 3-4 concurrent
- 96 GB VRAM: Below 128 GB minimum
- **Verdict: Below minimum requirements on both VRAM and main agent speed**

### Configuration B: 2x RTX 5090 -- "Performance Sweet Spot"
**Total VRAM**: 64 GB (well below 128 GB minimum)
**Estimated Cost**: CHF 7,000 - 10,000

| Component | Spec | Est. CHF |
|---|---|---|
| 2x RTX 5090 | 2x 32GB = 64GB | 4,800-7,000 |
| CPU | AMD Ryzen 9 9950X | 600 |
| Motherboard | X870E (PCIe 5.0) | 400 |
| RAM | 64GB DDR5 | 200 |
| PSU | 1,600W Titanium | 500 |
| Storage | 2TB NVMe | 150 |
| Case | Full tower | 200 |
| **Total** | | **~6,850 - 9,050** |

**Performance Estimate**:
- 70B Q4 (2x GPU, tensor parallel): ~27-40 tok/s -- **approaches minimum target of 30 tok/s**
- 8B sub-agents: Limited by only 2 GPUs -- both needed for 70B model
- Cannot run 70B + sub-agents + vision concurrently with only 64 GB
- **Verdict: Excellent per-card performance but insufficient VRAM for the multi-agent requirement**

### Configuration C: 4x RTX A6000 (used) -- "VRAM Maximizer"
**Total VRAM**: 192 GB (exceeds target!)
**Estimated Cost**: CHF 11,000 - 16,500

| Component | Spec | Est. CHF |
|---|---|---|
| 4x RTX A6000 (used) | 4x 48GB = 192GB | 7,600-13,200 |
| CPU | AMD Threadripper PRO 9955WX | 1,600 |
| Motherboard | ASUS WRX90E-SAGE | 1,000 |
| RAM | 128GB DDR5 ECC | 400 |
| PSU | 1,600W Platinum | 450 |
| Storage | 2TB NVMe | 150 |
| Case | Full tower | 200 |
| NVLink bridges (2x) | For 2 pairs | 100-200 |
| **Total** | | **~11,500 - 17,200** |

**Performance Estimate**:
- 70B FP8 (4x GPU, tensor parallel): ~20-30 tok/s -- **below minimum target**
- Can load 70B FP8 (~75GB) + 32B sub-agent model (~18GB) + vision model (~8GB) all in VRAM simultaneously
- Sub-agents at 15-20 tok/s on dedicated GPUs: achievable
- Concurrent streams: 4 GPUs allow flexible allocation
- Bandwidth bottleneck: 768 GB/s per A6000 is the weakest in this comparison
- **Verdict: Meets VRAM requirements spectacularly but may fall short on main agent speed. Only fits budget at lower used prices (~CHF 1,900-2,500/card). Budget risk is high.**

### Configuration D: 2x A100 80GB PCIe (used) + Cooling -- "HBM Bandwidth Play"
**Total VRAM**: 160 GB (meets target)
**Estimated Cost**: CHF 13,500 - 18,000

| Component | Spec | Est. CHF |
|---|---|---|
| 2x A100 80GB PCIe (used) | 2x 80GB = 160GB | 10,200-14,000 |
| CPU | AMD Ryzen 9 9950X | 600 |
| Motherboard | X870E or B850 | 300 |
| RAM | 64GB DDR5 | 200 |
| PSU | 1,000W Platinum | 250 |
| Storage | 2TB NVMe | 150 |
| Case + Aftermarket GPU cooling | Custom fans/cooling for passive GPUs | 400-600 |
| **Total** | | **~12,100 - 16,100** |

**Performance Estimate**:
- 70B FP8 (2x GPU, tensor parallel): ~30-40 tok/s -- **meets minimum target**
- HBM2e at 2,039 GB/s per card = combined 4,078 GB/s -- excellent bandwidth
- 160 GB fits 70B FP16 + sub-agent model + vision model
- Sub-agents: Can run concurrently using shared GPU memory (vLLM multi-model)
- Lower TDP (300W each) = easier cooling
- **Risk**: Passive cooling requires careful engineering; used A100s may be decommissioned enterprise hardware
- **Verdict: Strong option if you can find A100 80GB PCIe cards at ~CHF 5,100-5,500 each and solve the cooling. Likely exceeds budget at higher prices.**

### Configuration E: 3x RTX A6000 (used) + 1x RTX 5090 -- "Hybrid Approach"
**Total VRAM**: 176 GB (exceeds target)
**Estimated Cost**: CHF 10,500 - 14,500

| Component | Spec | Est. CHF |
|---|---|---|
| 1x RTX 5090 (new) | 32GB, 1,792 GB/s | 2,400-3,500 |
| 3x RTX A6000 (used) | 3x 48GB = 144GB | 5,700-9,900 |
| CPU | AMD Threadripper PRO 9955WX | 1,600 |
| Motherboard | ASUS WRX90E-SAGE | 1,000 |
| RAM | 128GB DDR5 ECC | 400 |
| PSU | 1,600W Platinum | 450 |
| Storage | 2TB NVMe | 150 |
| Case | Full tower | 200 |
| **Total** | | **~11,900 - 17,200** |

**Note**: This is an unusual configuration. Mixing GPU architectures adds complexity and may not be supported well by all inference engines. Not generally recommended.

### Configuration F: 2x RTX 5090 + 2x RTX 3090 -- "Speed + VRAM Budget Hybrid"
**Total VRAM**: 112 GB (below 128 GB minimum but close)
**Estimated Cost**: CHF 8,200 - 11,000

| Component | Spec | Est. CHF |
|---|---|---|
| 2x RTX 5090 (new) | 2x 32GB = 64GB | 4,800-7,000 |
| 2x RTX 3090 (used) | 2x 24GB = 48GB | 1,100-1,300 |
| CPU | AMD Threadripper PRO 9955WX | 1,600 |
| Motherboard | ASUS WRX90E-SAGE | 1,000 |
| RAM | 128GB DDR5 ECC | 400 |
| PSU | 1,800W Platinum | 500 |
| Storage | 2TB NVMe | 150 |
| Case | Full tower | 200 |
| **Total** | | **~9,750 - 12,150** |

**Performance Estimate**:
- 70B Q4 on 2x RTX 5090 (tensor parallel): ~27-40 tok/s
- Sub-agents on 2x RTX 3090: ~100 tok/s per GPU on 8B models
- Vision model on RTX 3090: concurrent with sub-agents
- Can run main agent + 2-4 sub-agents + vision concurrently
- **Verdict: Decent compromise. Main agent speed approaches minimum target. VRAM at 112GB is close to but below the 128GB minimum. Mixed architectures work fine for independent model serving.**

---

## 6. Recommendations

### The New Reality: MoE Models Change Everything

**The original analysis concluded that 60-80 tok/s on a 70B dense model was impossible within budget. This is no longer the relevant question.** The industry has shifted to MoE (Mixture of Experts) coding models, which only activate a fraction of parameters per token:

- **RTX PRO 6000 Blackwell** (single card, CHF 6,443): **134 tok/s** on GPT-OSS-120B (MoE) — far exceeds the 60-80 target
- **Mac Studio M3 Ultra** (CHF 6,100): **~69 tok/s** on GPT-OSS-120B (MoE) — meets the target

Dense 70B models remain at ~27-32 tok/s on the best prosumer hardware (RTX PRO 6000 or 2x RTX 5090). But with MoE models delivering equal or better coding quality at 4x the speed, dense 70B performance is now a fallback concern, not the primary metric.

**Practical targets achievable within budget:**
- MoE main agent (GPT-OSS-120B): 134 tok/s on RTX PRO 6000
- MoE sub-agents (Qwen3-30B-A3B): 150-200 tok/s on RTX PRO 6000
- Dense 70B fallback: 29-32 tok/s on RTX PRO 6000 (meets 30 tok/s minimum)

### Top 3 Recommended Configurations (Updated for MoE)

**RANK 1: Single RTX PRO 6000 Blackwell Build (~CHF 8,200)**
- **Best for**: Maximum value. MoE models make a single card sufficient.
- **VRAM**: 96 GB (single card)
- **MoE main agent**: ~134 tok/s — far exceeds target
- **Dense 70B fallback**: ~32 tok/s — meets minimum
- **Sub-agents**: Qwen3-Coder-30B-A3B at ~150-200 tok/s
- **Concurrent loading**: Main (65 GB) + sub-agent (18 GB) + vision (5 GB) = 88 GB, 8 GB free
- **Pros**: Quiet (300W), simple single-GPU build on AM5, NVLink upgrade path
- **Cons**: Shared bandwidth under heavy concurrency
- **Upgrade path**: Add 2nd PRO 6000 for 192 GB on Threadripper platform

**RANK 2: RTX PRO 6000 + RTX 5090 Build (~CHF 11,450)**
- **Best for**: Zero bandwidth contention between main agent and sub-agents
- **VRAM**: 128 GB (96 + 32)
- **MoE main agent on PRO 6000**: ~134 tok/s
- **Sub-agents on RTX 5090**: ~150-200 tok/s (dedicated GPU, no sharing)
- **Pros**: No bandwidth contention, can run largest MoE models (MiniMax-M2.5 at ~130 GB)
- **Cons**: More expensive, requires Threadripper platform

**RANK 3: 2x RTX 5090 (~CHF 7,090)**
- **Best for**: Budget-conscious build if PRO 6000 is too expensive
- **VRAM**: 64 GB (2x 32 GB) — limited
- **Dense 70B**: ~27-40 tok/s
- **MoE models**: Fast per-stream but VRAM limits concurrent loading
- **Pros**: Cheapest NVIDIA option, good per-card speed
- **Cons**: Only 64 GB VRAM constrains multi-model concurrent loading

**Previous recommendations (4x A6000, 4x RTX 3090, etc.) are superseded** — MoE models eliminate the need for massive VRAM and multi-GPU setups to hit speed targets. The RTX PRO 6000's combination of 96 GB VRAM + 1,792 GB/s bandwidth + 300W TDP is the clear winner.

### Meeting the 60-80 tok/s Target

**With MoE models, this target is already met:**

| Hardware | Model | tok/s | Meets Target? |
|---|---|---|---|
| RTX PRO 6000 (single) | GPT-OSS-120B (MoE) | **134** | **Far exceeds** |
| RTX PRO 6000 (single) | GPT-OSS-20B (MoE) | **185** | **Far exceeds** |
| Mac M3 Ultra | GPT-OSS-120B (MoE) | **~69** | **Yes** |
| RTX PRO 6000 (single) | Qwen2.5-Coder 72B (dense) | ~32 | Meets minimum only |
| 2x RTX 5090 | LLaMA 3.3 70B (dense) | ~27-40 | Meets minimum only |

For dense 70B models specifically, 60-80 tok/s remains achievable only on datacenter hardware (H100+). But this is no longer the relevant question — MoE coding models deliver comparable or better quality at 4x the speed.

---

## Sources

- [RTX 5090 Europe pricing](https://videocardz.com/newz/nvidia-geforce-rtx-5090-sees-first-price-drop-below-msrp-in-europe)
- [RTX 5090 Switzerland Digitec](https://www.digitec.ch/en/s1/producttype/graphics-card-106/filter/graphics-card-model-geforce-rtx-5090-45548)
- [RTX 5090 Toppreise.ch](https://www.toppreise.ch/browse?q=rtx+5090)
- [RTX 5090 LLM Benchmarks - RunPod](https://www.runpod.io/blog/rtx-5090-llm-benchmarks)
- [RTX 5090 bandwidth impact on LLMs](https://blog.neevcloud.com/the-impact-of-rtx-5090s-memory-bandwidth-on-llms)
- [Dual RTX 5090 vs H100 benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2)
- [Dual RTX 5090 beats H100](https://www.hardware-corner.net/dual-rtx-5090-vs-h100-for-llm/)
- [RTX 5090 specs and power](https://gamersnexus.net/gpus/nvidia-geforce-rtx-5090-founders-edition-review-benchmarks-gaming-thermals-power)
- [RTX 4090 pricing history](https://bestvaluegpu.com/en-eu/history/new-and-used-rtx-4090-price-history-and-specs/)
- [RTX 4090 inference benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx4090)
- [RTX 4090 no NVLink](https://www.windowscentral.com/hardware/computers-desktops/nvidia-kills-off-nvlink-on-rtx-4090)
- [RTX 3090 value king for AI](https://www.xda-developers.com/used-rtx-3090-value-king-local-ai/)
- [RTX 3090 NVLink review](https://www.servethehome.com/dual-nvidia-geforce-rtx-3090-nvlink-performance-review-asus-zotac/)
- [RTX 3090 inference benchmarks](https://www.ywian.com/blog/rtx-3090-ai-inference-benchmarks-2024)
- [Quad RTX 3090 workstation](https://customluxpcs.com/product/quad-rtx-3090/)
- [RTX A6000 inference benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-a6000)
- [4x A6000 vLLM benchmark](https://www.databasemart.com/blog/vllm-gpu-benchmark-a6000-4)
- [A6000 NVLink setup](https://massedcompute.com/faq-answers/?question=How+do+I+set+up+NVLink+for+multiple+RTX+A6000+GPUs+in+a+single+system?)
- [A100 80GB pricing forecast](https://levelupblogs.com/news/nvidia-a100-80gb-gpu-price-prediction-2026/)
- [A100 40GB vs 80GB](https://verda.com/blog/nvidia-a100-40gb-vs-80-gb)
- [H100 pricing guide](https://docs.jarvislabs.ai/blog/h100-price)
- [L40S specifications](https://www.fluence.network/blog/nvidia-l40s/)
- [L40S vs A100 comparison](https://acecloud.ai/blog/nvidia-l40s-vs-a100-ai-inference/)
- [GPU ranking for LLMs](https://www.hardware-corner.net/gpu-ranking-local-llm/)
- [Multi-GPU tensor parallelism guide](https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/)
- [llama.cpp multi-GPU breakthrough](https://medium.com/@jagusztinl/llama-cpp-performance-breakthrough-for-multi-gpu-setups-04c83a66feb2)
- [PCIe considerations for multi-GPU](https://aightbits.com/2025/04/15/diy-ai-pcie-considerations-for-multi-gpu-builds/)
- [NVLink vs PCIe for AI](https://www.hyperstack.cloud/blog/case-study/nvlink-vs-pcie-whats-the-difference-for-ai-workloads)
- [AMD Threadripper PRO 9000 pricing](https://www.techpowerup.com/339029/amd-announces-official-ryzen-threadripper-pro-9000-wx-series-pricing)
- [Dual RTX 5090 workstation approach - Puget Systems](https://www.pugetsystems.com/blog/2025/10/28/our-approach-to-dual-geforce-rtx-5090-workstations/)
- [ExLlamaV2 inference library](https://github.com/turboderp-org/exllamav2)
- [vLLM parallelism and scaling](https://docs.vllm.ai/en/stable/serving/parallelism_scaling/)
- [Best GPUs for LLM inference 2025](https://localllm.in/blog/best-gpus-llm-inference-2025)
