# RTX PRO 6000 Blackwell — Build Guide

> Last updated: February 16, 2026
> All prices verified against toppreise.ch on this date.

---

## Overview

A single NVIDIA RTX PRO 6000 Blackwell Workstation Edition (96 GB GDDR7) as the core of a local AI inference server. The build is designed to be **extensible**: a second GPU can be added later for 192 GB total VRAM.

**Key specs of the RTX PRO 6000:**
- 96 GB GDDR7 ECC, 512-bit bus
- 1,792 GB/s memory bandwidth
- 300W TDP (quiet, standard outlet)
- NVLink support (for pairing two cards)
- PCIe 5.0 x16

**Performance (single card, Ollama):**

| Model | Type | tok/s |
|-------|------|-------|
| GPT-OSS-120B | MoE (120B total / ~20B active) | **134** |
| GPT-OSS-20B | MoE (20B total / ~5B active) | **185** |
| Qwen3-Coder-30B-A3B | MoE | **~150-200** |
| LLaMA 3.3 70B | Dense | 32 |
| Qwen 2.5 72B | Dense | 29 |

96 GB fits: main agent (~65 GB) + sub-agent (~18 GB) + vision model (~5 GB) = 88 GB, with 8 GB free.

---

## Platform Choice: AM5 vs. Threadripper

Two platform options are provided. Both use the same GPU, storage, and RAM. The difference is the CPU/motherboard, which determines how well a **second GPU** works if added later.

### When does the platform matter?

During inference, each GPU reads from its own VRAM at 1,792 GB/s. The PCIe connection between CPU and GPU is only used for:
1. **Loading models** from SSD → GPU VRAM (one-time at startup, takes seconds either way)
2. **Inter-GPU communication** when splitting one model across 2 GPUs

| Scenario (with 2 GPUs) | AM5 (x8/x8, no NVLink) | Threadripper (x16/x16 + NVLink) |
|-------------------------|------------------------|-------------------------------|
| **Each GPU runs its own model** (main agent on GPU 1, sub-agents on GPU 2) | Works perfectly — GPUs don't talk to each other | Same — no difference |
| **One large model split across both GPUs** (e.g. a 150 GB dense model) | Slower inter-GPU transfer (~16 GB/s per direction via PCIe 5.0 x8) | Much faster (~112 GB/s via NVLink) |

**Bottom line:** If the second GPU will run independent models (sub-agents, vision), AM5 is fine. If you plan to split single models larger than 96 GB across both GPUs, Threadripper with NVLink is significantly better.

### Platform comparison

| | AM5 | Threadripper (TRX50) |
|--|-----|---------------------|
| CPU | Ryzen 9 9950X (16C/32T, 170W) | Threadripper 7960X (24C/48T, 350W) |
| PCIe lanes from CPU | 24x PCIe 5.0 | 48x PCIe 5.0 |
| With 1 GPU | x16 (full bandwidth) | x16 (full bandwidth) |
| With 2 GPUs | x8/x8 (half bandwidth per GPU) | x16/x16 (full bandwidth) |
| NVLink between GPUs | Not supported | Supported |
| Platform cost | ~CHF 709 | ~CHF 1,619 |
| **Difference** | — | **+CHF 910** |

---

## Option A: AM5 Platform (~CHF 9,690)

The cheaper build. Upgrade to 2 GPUs is possible with reduced inter-GPU bandwidth (fine for independent models per GPU).

### Parts list

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPU** | [NVIDIA RTX PRO 6000 Blackwell WS (96 GB)](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | 7,691 |
| 2 | **CPU** | [AMD Ryzen 9 9950X (16C/32T, 4.3 GHz)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-9-9950X-Granite-Ridge-16x-4-3GHz-100-100001277WOF-p771475) | 440 |
| 3 | **Motherboard** | [ASUS TUF Gaming X870E-Plus WIFI7](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-TUF-GAMING-X870E-PLUS-WIFI7-AMD-X870E-90MB1M70-M0EAY0-p809177) | 269 |
| 4 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 5 | **PSU** | [be quiet! Pure Power 12 M 1000W (80+ Gold)](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Pure-Power-12-M-1000-Watts-BN345-p717884) | 171 |
| 6 | **Case** | [Fractal Design Define 7 Black (solid panel)](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-Black-FD-C-DEF7A-01-p601634) | 139 |
| 7 | **CPU Cooler** | [Noctua NH-D15](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-D15-p366609) | 98 |
| 8 | **Storage** | [Samsung 990 Pro 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-PRO-NVMe-M-2-SSD-2TB-MZ-V9P2T0BW-p702884) | 203 |
| 9 | **Misc** | Cables, thermal paste | 50 |
| | **TOTAL** | | **~CHF 9,690** |

### Power and noise

- **Power draw:** ~475W under full GPU load (300W GPU + 125W CPU + 50W rest). The 1000W PSU has ample headroom.
- **Noise:** Quiet. The PRO 6000 blower at 300W is far quieter than consumer cards at 450-575W. Noctua + Fractal Define 7 (sound-dampened) keeps system noise low.
- **Electrical:** Standard Swiss 10A/230V outlet (2,300W capacity) — no issues.

### Upgrade path: adding a 2nd GPU

To add a second RTX PRO 6000 later:

| Item | Action | Est. Cost |
|------|--------|-----------|
| 2nd GPU | Add RTX PRO 6000 Blackwell | ~CHF 7,691 |
| PSU | Upgrade to 1200W+ (600W GPU + 125W CPU + 50W = ~775W) | ~CHF 200-300 |
| **Total upgrade cost** | | **~CHF 7,890-7,990** |

**Caveats:**
- The 2nd GPU runs at PCIe 5.0 x8 (the x16 slot is shared/bifurcated). This is ~16 GB/s per direction — fine for independent models, but a bottleneck for tensor parallelism across GPUs.
- No NVLink on AM5 — the two GPUs cannot be linked for unified memory.
- Verify that the ASUS TUF X870E-Plus WIFI7 supports x8/x8 bifurcation in BIOS before purchasing. If not, the 2nd slot may run at PCIe 4.0 x4 from the chipset (much slower). Consider a higher-end X870E board with confirmed bifurcation support if this is a concern.

---

## Option B: Threadripper Platform (~CHF 10,850)

The future-proof build. Adding a 2nd GPU is plug-and-play with full bandwidth and NVLink.

### Parts list

| # | Component | Specification | Price (CHF) |
|---|-----------|--------------|-------------|
| 1 | **GPU** | [NVIDIA RTX PRO 6000 Blackwell WS (96 GB)](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | 7,691 |
| 2 | **CPU** | [AMD Threadripper 7960X (24C/48T, 4.2 GHz)](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329) | 971 |
| 3 | **Motherboard** | [ASUS Pro WS TRX50-SAGE WIFI (CEB, 4x PCIe 5.0 x16)](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606) | 648 |
| 4 | **RAM** | [Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB)](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | 629 |
| 5 | **PSU** | [be quiet! Dark Power Pro 13 1600W (80+ Titanium)](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Dark-Power-Pro-13-1600-Watts-BN332-p729058) | 352 |
| 6 | **Case** | [Fractal Design Define 7 XL Black (solid panel)](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-XL-Black-FD-C-DEF7X-01-p601653) | 193 |
| 7 | **CPU Cooler** | [Noctua NH-U14S TR5-SP6 (sTR5 socket)](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500) | 111 |
| 8 | **Storage** | [Samsung 990 Pro 2TB NVMe](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-PRO-NVMe-M-2-SSD-2TB-MZ-V9P2T0BW-p702884) | 203 |
| 9 | **Misc** | Cables, thermal paste | 50 |
| | **TOTAL** | | **~CHF 10,848** |

### Power and noise

- **Power draw:** ~525W under full GPU load (300W GPU + 175W CPU + 50W rest). The 1600W PSU is massively oversized for 1 GPU — this is intentional for the 2-GPU upgrade.
- **Noise:** Quiet with 1 GPU. The Threadripper draws more than the 9950X but is still well within what Noctua + Define 7 XL can handle silently.
- **Electrical:** Standard Swiss outlet — no issues.

### Why the larger components

- **1600W PSU:** Sized for 2x PRO 6000 (600W GPU + 175W CPU + 75W rest = ~850W). Buying it now avoids replacing the PSU later.
- **Define 7 XL:** The TRX50-SAGE is a CEB form factor board (larger than ATX). The XL case guarantees fitment and provides better airflow for 2 GPUs.
- **4 DIMM slots + 2 empty:** The TRX50-SAGE has 4 DDR5 slots. 2x32 GB leaves 2 slots free to expand to 128 GB later if needed.

### Upgrade path: adding a 2nd GPU

To add a second RTX PRO 6000 later:

| Item | Action | Est. Cost |
|------|--------|-----------|
| 2nd GPU | Add RTX PRO 6000 Blackwell | ~CHF 7,691 |
| NVLink bridge | Optional — enables 112.5 GB/s inter-GPU link | ~CHF 100-200 |
| **Total upgrade cost** | | **~CHF 7,791-7,891** |

**No other changes needed.** The PSU, case, motherboard, and cooling are already sized for 2 GPUs. Just install the card, optionally add the NVLink bridge, and you have 192 GB VRAM with full x16/x16 bandwidth.

**With 2x PRO 6000 (192 GB):**
- Power draw: ~850W (well within 1600W PSU)
- Fits standard Swiss 10A outlet (850W < 2,300W)
- Can run the largest MoE models (e.g. MiniMax-M2.5 at ~130 GB) entirely in VRAM
- Main agent + multiple sub-agents + vision, each with dedicated VRAM, zero contention

---

## Side-by-side comparison

| | Option A (AM5) | Option B (Threadripper) |
|--|---------------|------------------------|
| **Total cost (1 GPU)** | **~CHF 9,690** | **~CHF 10,848** |
| **Cost to add 2nd GPU** | ~CHF 7,890-7,990 (GPU + new PSU) | ~CHF 7,791-7,891 (GPU + NVLink bridge) |
| **Total cost (2 GPUs)** | ~CHF 17,580-17,680 | ~CHF 18,640-18,740 |
| Performance (1 GPU) | Identical | Identical |
| Performance (2 GPUs, independent models) | Same — no difference | Same — no difference |
| Performance (2 GPUs, split model) | Slower (PCIe x8, no NVLink) | Faster (PCIe x16 + NVLink) |
| GPU upgrade process | Add GPU + replace PSU + verify BIOS bifurcation | Just add GPU |
| PCIe slots for future expansion | 1x free x16 (after 2 GPUs: none) | 2x free x16 (after 2 GPUs: 2 remain) |
| Noise (1 GPU) | Very quiet | Very quiet |
| RAM expandability | Up to 128 GB (4 slots, 2 used) | Up to 128 GB (4 slots, 2 used) |

---

## Shop links (Switzerland)

All links verified February 16, 2026.

### GPU

| Product | Price (CHF) | Link |
|---------|-------------|------|
| **RTX PRO 6000 Blackwell WS** (NVIDIA OEM — cheapest) | 7,691 | [toppreise.ch](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) |
| RTX PRO 6000 (PNY retail) | 7,848+ | [toppreise.ch](https://www.toppreise.ch/price-comparison/Graphics-cards/PNY-Nvidia-RTX-Pro-6000-Blackwell-WS-Nvidia-VCNRTXPRO6000-PB-p804716) |
| RTX PRO 6000 (PNY, digitec) | 8,744 | [digitec.ch](https://www.digitec.ch/en/s1/product/pny-rtx-pro-6000-blackwell-workstation-edition-96-gb-graphics-card-56565277) |
| RTX PRO 6000 (NVIDIA bulk, galaxus) | varies | [galaxus.ch](https://www.galaxus.ch/en/s1/product/nvidia-rtx-pro-6000-blackwell-works-bulk-96-gb-graphics-card-58324439) |
| RTX PRO 6000 (brack) | varies | [brack.ch](https://www.brack.ch/pny-grafikkarte-nvidia-rtx-pro-6000-blackwell-ws-96-gb-1868148) |

> **Buy the NVIDIA OEM version** (SKU 900-5G144-2200-000) via toppreise.ch for CHF 7,691. The PNY retail box is the same GPU at CHF 7,848+ — the difference is packaging and branding only.

### Option A components (AM5)

| Component | Price (CHF) | Toppreise | Digitec | Brack |
|-----------|-------------|-----------|---------|-------|
| AMD Ryzen 9 9950X | 440 | [toppreise](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-9-9950X-Granite-Ridge-16x-4-3GHz-100-100001277WOF-p771475) | [digitec](https://www.digitec.ch/de/s1/product/amd-ryzen-9-9950x-am5-430-ghz-16-core-prozessor-47373754) | [brack](https://www.brack.ch/amd-cpu-ryzen-9-9950x-4-3-ghz-1753689) |
| ASUS TUF X870E-Plus WIFI7 | 269 | [toppreise](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-TUF-GAMING-X870E-PLUS-WIFI7-AMD-X870E-90MB1M70-M0EAY0-p809177) | [digitec](https://www.digitec.ch/de/s1/product/asus-tuf-gaming-x870e-plus-wifi7-am5-amd-x870e-atx-mainboard-58291944) | [brack](https://www.brack.ch/fr/asus-carte-mere-tuf-gaming-x870e-plus-wifi7-1891498) |
| Noctua NH-D15 | 98 | [toppreise](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-D15-p366609) | [digitec](https://www.digitec.ch/en/s1/product/noctua-nh-d15-165-mm-cpu-coolers-2580255) | [brack](https://www.brack.ch/noctua-cpu-kuehler-nh-d15-300317) |
| be quiet! Pure Power 12 M 1000W | 171 | [toppreise](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Pure-Power-12-M-1000-Watts-BN345-p717884) | [digitec](https://www.digitec.ch/en/s1/product/be-quiet-pure-power-12-m-1000-w-pc-netzteil-24077995) | [brack](https://www.brack.ch/be-quiet-netzteil-pure-power-12-m-1000-w-1520808) |
| Fractal Design Define 7 Black | 139 | [toppreise](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-Black-FD-C-DEF7A-01-p601634) | [digitec](https://www.digitec.ch/de/s1/product/fractal-define-7-black-solid-atx-matx-mini-itx-e-atx-pc-gehaeuse-12757901) | [brack](https://www.brack.ch/fractal-design-pc-gehaeuse-define-7-schwarz-1288894) |

### Option B components (Threadripper)

| Component | Price (CHF) | Toppreise | Digitec | Brack |
|-----------|-------------|-----------|---------|-------|
| AMD Threadripper 7960X | 971 | [toppreise](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329) | — | — |
| ASUS Pro WS TRX50-SAGE WIFI | 648 | [toppreise](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606) | — | — |
| Noctua NH-U14S TR5-SP6 | 111 | [toppreise](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500) | — | — |
| be quiet! Dark Power Pro 13 1600W | 352 | [toppreise](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Dark-Power-Pro-13-1600-Watts-BN332-p729058) | — | — |
| Fractal Design Define 7 XL Black | 193 | [toppreise](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-XL-Black-FD-C-DEF7X-01-p601653) | — | — |

### Shared components (both options)

| Component | Price (CHF) | Toppreise | Digitec | Brack |
|-----------|-------------|-----------|---------|-------|
| Kingston FURY Beast DDR5-5600 64 GB CL40 (2x32 GB) | 629 | [toppreise](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-CL40-Black-KF556C40BBK2-64-p688007) | [digitec](https://www.digitec.ch/en/s1/product/kingston-fury-beast-2-x-32gb-5600-mhz-ddr5-ram-dimm-ram-20929817) | [brack](https://www.brack.ch/kingston-ddr5-ram-fury-beast-5600-mhz-2x-32-gb-1509447) |
| Samsung 990 Pro 2TB NVMe | 203 | [toppreise](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-PRO-NVMe-M-2-SSD-2TB-MZ-V9P2T0BW-p702884) | [digitec](https://www.digitec.ch/en/s1/product/samsung-990-pro-2000-gb-m2-2280-ssd-22097721) | [brack](https://www.brack.ch/samsung-ssd-990-pro-m-2-2280-nvme-2000-gb-1425209) |

---

## Notes

### Why not the RTX 5090 as second GPU?

The Config 2.0b in the main research doc pairs a PRO 6000 with an RTX 5090 (32 GB) for CHF 11,450 on Threadripper. This is a valid option if:
- You want a cheaper second GPU (~CHF 2,080-2,340 vs CHF 7,691)
- You only need the 2nd GPU for sub-agents (32 GB is enough for smaller models)

The trade-off: no NVLink between a PRO 6000 and a 5090 (different GPU architectures for NVLink pairing). They'd communicate via PCIe only. For independent models per GPU, this doesn't matter.

### RAM speed

DDR5 speed has negligible impact on inference performance. Models live in GPU VRAM — system RAM is only used for loading models at startup and OS overhead. CL40 DDR5-5600 is the cheapest option and works perfectly. Do not overspend on RAM speed.

### Operating system

Ubuntu Server (free) is the recommended OS. Headless, accessed via SSH/API. NVIDIA CUDA drivers have first-class Linux support. No Windows license needed.
