# Build Guides — Parts Lists, Prices & Shop Links

> Last updated: February 17, 2026. All prices verified against toppreise.ch.

For the rationale behind each configuration, see [decision-summary.md](decision-summary.md). For GPU deep-dives, see [gpu-research-nvidia.md](gpu-research-nvidia.md).

---

## Config #1: Single RTX PRO 6000 (RECOMMENDED)

A single NVIDIA RTX PRO 6000 Blackwell Workstation Edition (96 GB GDDR7) as the core of a local AI inference server. Extensible: a second GPU can be added later for 192 GB total VRAM.

**Performance (single card, Ollama):**

| Model | Type | tok/s |
|-------|------|-------|
| GPT-OSS-120B | MoE (120B total / ~20B active) | **134** |
| Qwen3-Coder-30B-A3B | MoE | **~150-200** |
| LLaMA 3.3 70B | Dense | 32 |

96 GB fits: main agent (~65 GB) + sub-agent (~18 GB) + vision model (~5 GB) = 88 GB, with 8 GB free.

### Platform Choice: AM5 vs. Threadripper

Both use the same GPU, storage, and RAM. The difference is how well a **second GPU** works if added later.

| Scenario (with 2 GPUs) | AM5 (x8/x8, no NVLink) | Threadripper (x16/x16 + NVLink) |
|-------------------------|------------------------|-------------------------------|
| **Each GPU runs its own model** (main on GPU 1, sub-agents on GPU 2) | Works perfectly — GPUs don't talk to each other | Same — no difference |
| **One large model split across both GPUs** (e.g. a 150 GB dense model) | Slower inter-GPU transfer (~16 GB/s per direction via PCIe 5.0 x8) | Much faster (~112 GB/s via NVLink) |

**Bottom line:** If the second GPU will run independent models, AM5 is fine. If you plan to split single models larger than 96 GB across both GPUs, Threadripper with NVLink is significantly better.

| | AM5 | Threadripper (TRX50) |
|--|-----|---------------------|
| CPU | Ryzen 9 9950X (16C/32T, 170W) | Threadripper 7960X (24C/48T, 350W) |
| PCIe lanes | 24x PCIe 5.0 | 48x PCIe 5.0 |
| With 2 GPUs | x8/x8 (half bandwidth per GPU) | x16/x16 (full bandwidth) |
| NVLink | Not supported | Supported |
| Platform cost | ~CHF 628 | ~CHF 1,608 |

---

### Option A: AM5 Platform (~CHF 9,485)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPU** | [NVIDIA RTX PRO 6000 Blackwell WS (96 GB)](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | 7,691 |
| 2 | **CPU** | [AMD Ryzen 9 9950X (16C/32T, 4.3 GHz)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-9-9950X-Granite-Ridge-16x-4-3GHz-100-100001277WOF-p771475) | 439 |
| 3 | **Motherboard** | [MSI X870E Gaming Plus WIFI (ATX, PCIe 5.0 x16)](https://www.toppreise.ch/price-comparison/Motherboards/MSI-X870E-GAMING-PLUS-WIFI-AMD-X870E-7E70-001R-p809245) | 189 |
| 4 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 5 | **PSU** | [Enermax Revolution III 1000W (80+ Gold, ATX 3.1)](https://www.toppreise.ch/price-comparison/Power-supplies/ENERMAX-REVOLUTION-III-1000-Watts-ERV1000G-AHG-MAC-p798249) | 91 |
| 6 | **Case** | [Fractal Design Define 7 Black (solid panel)](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-Black-FD-C-DEF7A-01-p601634) | 137 |
| 7 | **CPU Cooler** | [be quiet! Dark Rock Pro 5](https://www.toppreise.ch/price-comparison/CPU-coolers/BE-QUIET-Dark-Rock-Pro-5-BK036-p748972) | 76 |
| 8 | **Storage** | [Samsung 990 EVO Plus 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-EVO-Plus-SSD-M-2-2TB-MZ-V9S2T0BW-p784700) | 183 |
| 9 | **Misc** | Cables, thermal paste | 50 |
| | **TOTAL** | | **~CHF 9,485** |

**Power:** ~475W under full GPU load. 1000W PSU has ample headroom.
**Noise:** Quiet. PRO 6000 blower at 300W + Dark Rock Pro 5 + Fractal Define 7 (sound-dampened).
**Electrical:** Standard Swiss 10A/230V outlet (2,300W capacity) — no issues.

**Upgrade path (add 2nd GPU):** Add RTX PRO 6000 (~CHF 7,691) + upgrade PSU to 1200W+ (~CHF 200-300). Total upgrade: ~CHF 7,890-7,990. Caveats: 2nd GPU runs at PCIe 5.0 x8 (fine for independent models). No NVLink on AM5.

---

### Option B: Threadripper Platform (~CHF 10,666)

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPU** | [NVIDIA RTX PRO 6000 Blackwell WS (96 GB)](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | 7,691 |
| 2 | **CPU** | [AMD Threadripper 7960X (24C/48T, 4.2 GHz)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329) | 966 |
| 3 | **Motherboard** | [ASUS Pro WS TRX50-SAGE WIFI (CEB, 4x PCIe 5.0 x16)](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606) | 642 |
| 4 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 5 | **PSU** | [ASRock Phantom Gaming PG-1600G 1600W (80+ Gold, ATX 3.1)](https://www.toppreise.ch/price-comparison/Power-supplies/ASROCK-Phantom-Gaming-PG-1600G-1600-Watts-90-UXP160-GFEAAB-p792444) | 205 |
| 6 | **Case** | [Fractal Design Define 7 XL Black (solid panel)](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-XL-Black-FD-C-DEF7X-01-p601653) | 196 |
| 7 | **CPU Cooler** | [Noctua NH-U14S TR5-SP6 (sTR5 socket)](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500) | 104 |
| 8 | **Storage** | [Samsung 990 EVO Plus 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-EVO-Plus-SSD-M-2-2TB-MZ-V9S2T0BW-p784700) | 183 |
| 9 | **Misc** | Cables, thermal paste | 50 |
| | **TOTAL** | | **~CHF 10,666** |

**Power:** ~525W under full GPU load. 1600W PSU is oversized for 1 GPU — sized for 2-GPU upgrade.
**Noise:** Quiet with 1 GPU.
**Why the larger components:** 1600W PSU avoids replacement when adding 2nd GPU. Define 7 XL fits the CEB motherboard and provides airflow for 2 GPUs. 4 DIMM slots (2 used) leave room for 128 GB later.

**Upgrade path (add 2nd GPU):** Add RTX PRO 6000 (~CHF 7,691) + optional NVLink bridge (~CHF 100-200). Total upgrade: ~CHF 7,791-7,891. **No other changes needed** — PSU, case, motherboard, and cooling are already sized for 2 GPUs.

**With 2x PRO 6000 (192 GB):** Power draw ~850W (well within 1600W PSU and standard Swiss outlet). Can run the largest MoE models (MiniMax-M2.5 at ~130 GB) entirely in VRAM.

---

### Side-by-side comparison

| | Option A (AM5) | Option B (Threadripper) |
|--|---------------|------------------------|
| **Total cost (1 GPU)** | **~CHF 9,485** | **~CHF 10,666** |
| **Cost to add 2nd GPU** | ~CHF 7,890-7,990 (GPU + new PSU) | ~CHF 7,791-7,891 (GPU + NVLink bridge) |
| **Total cost (2 GPUs)** | ~CHF 17,375-17,475 | ~CHF 18,457-18,557 |
| Performance (1 GPU) | Identical | Identical |
| Performance (2 GPUs, independent models) | Same | Same |
| Performance (2 GPUs, split model) | Slower (PCIe x8, no NVLink) | Faster (PCIe x16 + NVLink) |
| GPU upgrade process | Add GPU + replace PSU + verify BIOS | Just add GPU |

---

## Config #2: RTX PRO 6000 + RTX 5090 (~CHF 11,450)

For zero bandwidth contention between main agent and sub-agents, or running the largest MoE models.

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPU 1** | [NVIDIA RTX PRO 6000 Blackwell WS (96 GB)](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | 7,691 |
| 2 | **GPU 2** | [NVIDIA RTX 5090 (32 GB)](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250) | ~2,224 |
| 3 | **CPU** | [AMD Threadripper 7960X (24C/48T)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329) | 966 |
| 4 | **Motherboard** | [ASUS Pro WS TRX50-SAGE WIFI](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606) | 642 |
| 5 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 6 | **PSU** | [ASRock Phantom Gaming PG-1600G 1600W](https://www.toppreise.ch/price-comparison/Power-supplies/ASROCK-Phantom-Gaming-PG-1600G-1600-Watts-90-UXP160-GFEAAB-p792444) | 205 |
| 7 | **Case** | [Fractal Design Define 7 XL Black](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-XL-Black-FD-C-DEF7X-01-p601653) | 196 |
| 8 | **CPU Cooler** | [Noctua NH-U14S TR5-SP6](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500) | 104 |
| 9 | **Storage** | [Samsung 990 EVO Plus 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-EVO-Plus-SSD-M-2-2TB-MZ-V9S2T0BW-p784700) | 183 |
| 10 | **Misc** | Cables, fans | 100 |
| | **TOTAL** | | **~CHF 11,940** |

**VRAM:** 128 GB (96 + 32)
**Power:** ~875W under full load (300W + 575W GPUs + system).
**Noise:** Moderate — PRO 6000 blower + RTX 5090 open-air.

**Strategy:** Main agent (GPT-OSS-120B) on PRO 6000 at ~134 tok/s. Sub-agents (Qwen3-Coder-30B-A3B) on RTX 5090 at ~150-200 tok/s. Zero bandwidth contention.

**No NVLink** between PRO 6000 and 5090 (different architectures). They communicate via PCIe only — fine for independent models per GPU.

**Upgrade path:** Replace RTX 5090 with 2nd PRO 6000 for 192 GB with NVLink.

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

---

## Config #4: 2x RTX 5090 (~CHF 7,665)

Budget NVIDIA build. Fast per-stream but VRAM-limited for concurrent models.

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPUs** | [2x NVIDIA RTX 5090 (32 GB each)](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250) | 2 x ~2,350 = ~4,700 |
| 2 | **CPU** | [AMD Threadripper 7960X (24C/48T)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329) | 966 |
| 3 | **Motherboard** | [ASUS Pro WS TRX50-SAGE WIFI](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606) | 642 |
| 4 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 5 | **PSU** | [ASRock Phantom Gaming PG-1600G 1600W](https://www.toppreise.ch/price-comparison/Power-supplies/ASROCK-Phantom-Gaming-PG-1600G-1600-Watts-90-UXP160-GFEAAB-p792444) | 205 |
| 6 | **Case** | [Fractal Design Define 7 XL Black](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-XL-Black-FD-C-DEF7X-01-p601653) | 196 |
| 7 | **CPU Cooler** | [Noctua NH-U14S TR5-SP6](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500) | 104 |
| 8 | **Storage** | [Samsung 990 EVO Plus 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-EVO-Plus-SSD-M-2-2TB-MZ-V9S2T0BW-p784700) | 183 |
| 9 | **Misc** | Cables, fans | 100 |
| | **TOTAL** | | **~CHF 7,725** |

**VRAM:** 64 GB (2x 32 GB) — well below 96 GB practical minimum.
**Power:** ~1,350W under full load (2x 575W GPUs + system).
**Noise:** Moderate-loud (two 575W consumer GPUs).

**Performance:** 70B Q4 (2 GPUs): ~27-40 tok/s. 32B Q4 (1 GPU): ~61 tok/s.
**Concurrent models:** Tight — 70B Q4 (~43 GB) fills both GPUs; ~21 GB left for everything else.

**Why it's ranked #4:** The 64 GB VRAM is a hard constraint for multi-agent workflows. Best suited if you exclusively run models under 32 GB per GPU.

---

## GPU Shop Links (Switzerland)

All links verified February 17, 2026.

| Product | Price (CHF) | Link |
|---------|-------------|------|
| **RTX PRO 6000 Blackwell WS** (NVIDIA OEM — cheapest) | 7,691 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) |
| RTX PRO 6000 (PNY retail) | 7,848+ | [toppreise.ch](https://www.toppreise.ch/price-comparison/Graphics-cards/PNY-Nvidia-RTX-Pro-6000-Blackwell-WS-Nvidia-VCNRTXPRO6000-PB-p804716) |
| RTX PRO 6000 (digitec) | 8,744 | [digitec.ch](https://www.digitec.ch/en/s1/product/pny-rtx-pro-6000-blackwell-workstation-edition-96-gb-graphics-card-56565277) |
| RTX PRO 6000 (galaxus bulk) | varies | [galaxus.ch](https://www.galaxus.ch/en/s1/product/nvidia-rtx-pro-6000-blackwell-works-bulk-96-gb-graphics-card-58324439) |
| RTX 5090 (from ~2,224) | varies | [toppreise.ch](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250) |

> **Buy the NVIDIA OEM RTX PRO 6000** (SKU 900-5G144-2200-000) via toppreise.ch for CHF 7,691. The PNY retail box is the same GPU at CHF 7,848+ — the difference is packaging only.

---

## Notes

### RAM speed

DDR5 speed has negligible impact on inference performance. Models live in GPU VRAM — system RAM is only used for loading models at startup and OS overhead. DDR5-5600 CL40 is the cheapest option and works perfectly. Do not overspend on RAM speed.

### Operating system

Ubuntu Server (free) is the recommended OS. Headless, accessed via SSH/API. NVIDIA CUDA drivers have first-class Linux support. No Windows license needed.

### Why not the RTX 5090 as second GPU in Config #1?

Config #2 pairs a PRO 6000 with an RTX 5090 on Threadripper. This is valid if you want a cheaper second GPU (~CHF 2,224 vs CHF 7,691) and only need 32 GB for sub-agents. Trade-off: no NVLink between PRO 6000 and 5090 (different architectures). For independent models per GPU, this doesn't matter.
