# AI Inference Hardware — Workstation & Build Options

> Research date: February 16, 2026
> Budget: 4,000 - 15,000 CHF | Target: 96-256 GB VRAM (MoE models reduce VRAM needs) | Home office (noise-sensitive) | Headless server via SSH/API

---

## Table of Contents

1. [Pre-Built AI Workstation Vendors](#1-pre-built-ai-workstation-vendors)
2. [DIY Build Configurations](#2-diy-build-configurations)
3. [Base System Component Analysis](#3-base-system-component-analysis)
4. [Configuration Cost Summaries](#4-configuration-cost-summaries)
5. [Recommendations](#5-recommendations)

---

## 1. Pre-Built AI Workstation Vendors

### 1.1 BIZON Tech (bizon-tech.com) — US-based, ships worldwide

**Relevant Models:**

| Model | GPUs | CPU Platform | Starting Price (USD) |
|-------|------|-------------|---------------------|
| **G3000** | 2-4 GPUs | Intel Xeon W (8-60 cores) | $4,437 |
| **X5500 G2** | 2-4 GPUs | AMD Threadripper PRO 9000WX/7000WX | $5,990 |
| **Z5000 G2** | 4-6 GPUs (liquid-cooled) | Intel Xeon W | $22,055+ |
| **G7000 G4** | Up to 8 GPUs | Dual Intel Xeon Scalable | $15,333 |

**GPU Options & Add-on Pricing (per GPU):**
- RTX 5090 (32 GB): +$4,500
- RTX 5080 (16 GB): +$2,103
- RTX 6000 Ada (48 GB): +$8,749
- NVIDIA A100 (80 GB): +$23,853
- NVIDIA H100 (94 GB): +$34,846

**Example Configurations (estimated total USD):**
- G3000 + 2x RTX 5090 + 24-core Xeon W: ~$14,000-16,000
- G3000 + 2x RTX 4090 (if still available): ~$10,000-12,000
- X5500 G2 + 4x RTX 5090: ~$25,000+ (over budget)

**Shipping to Switzerland:**
- Ships worldwide via FedEx/DHL/UPS (5-9 business days, or 2-5 express)
- Import duties, VAT (7.7% Swiss), and customs charges are buyer's responsibility
- Expect ~8-12% additional cost for Swiss import (VAT + customs + brokerage)

**Warranty:**
- 3-year standard (included), 4-year (+8% of price), 5-year (+15% of price)
- International warranty: customer pays shipping both ways for repairs
- "Lifetime Expert Care" support included

**Noise:** Water-cooled models (Z-series) rated ~43 dB idle. Air-cooled G3000 will be louder.

**Assessment for your budget:**
A BIZON G3000 with 2x RTX 5090 (64 GB VRAM) lands around $14,000-16,000 USD (~CHF 12,700-14,500) before Swiss import costs, which pushes it to the top or slightly over the CHF 15,000 budget. The 4-GPU configs are well over budget. Still a viable option if you want a turnkey 2x RTX 5090 system.

---

### 1.2 Joule Performance (jouleperformance.com) — Swiss-based

**Assessment: Not suitable for this use case.**

Joule Performance is a Swiss PC builder focused on gaming PCs and entry-level workstations. Their current workstation lineup:

| Model | CPU | GPU | RAM | Price (CHF) |
|-------|-----|-----|-----|-------------|
| ProArt RTX3050 TR | Threadripper 7960X | RTX 3050 6GB | 32 GB | CHF 3,712 |
| ProArt RTX3050 R5 | Ryzen 5 7600X | RTX 3050 6GB | 32 GB | CHF 2,177 |
| AXXIV R5 | Ryzen 5 7500F | RTX 3050 6GB | 32 GB | CHF 1,764 |
| AXXIV I5 | Core i5 14400 | RTX 3050 6GB | 32 GB | CHF 2,015 |

- No multi-GPU configurations offered
- No AI/ML-specific workstations
- Maximum single GPU (RTX 3050 with 6 GB VRAM)
- They have a custom PC configurator, but it is oriented toward gaming/creative work

**Verdict:** Does not offer what you need. Their workstations are basic ISV-certified machines for CAD/video editing with very low VRAM. Could potentially custom-order through their configurator, but they don't specialize in multi-GPU AI workloads.

---

### 1.3 Lambda Labs — No longer sells hardware

**Lambda has discontinued on-premise hardware sales as of August 29, 2025.**

Their Vector, Vector One, and Vector Pro workstations are no longer manufactured or sold. Lambda has pivoted entirely to cloud GPU services. Existing warranties will still be honored.

**Verdict:** Not available. Lambda is cloud-only now.

---

### 1.4 Puget Systems — US only (no European shipping)

Puget Systems offers excellent multi-GPU AI workstations:
- Multi-GPU Rackmount: up to 2x RTX 5090 or 4x RTX 6000 Max-Q
- Single-GPU workstations for generative AI
- Pricing ranges from ~$3,000 to $60,000+

**However: Puget Systems only ships to the US and Canada.** They explicitly do not support international shipping to Europe or Switzerland, and warranty service is not available outside US/Canada.

**Verdict:** Not available for Swiss buyers unless you have a US shipping address and are willing to self-import with no warranty support.

---

### 1.5 BRESSNER Technology (shop.bressner.de) — German, ships to EU/CH

BRESSNER is a German company specializing in GPU servers and HPC solutions.

**Offerings:**
- GPU servers in 1U-4U rack form factors, up to 10 GPU slots
- Support for NVIDIA RTX 5090, RTX 4090, Tesla, and professional GPUs
- Liquid cooling and immersion cooling options
- Custom-configured systems (CPU, RAM, GPU to spec)
- 24-hour burn-in test before delivery

**Pricing:** Not publicly listed — all systems are configure-to-order. Contact required (vertrieb@bressner.de or +49 8142 47284-70).

**Shipping:** German company, ships within EU. Switzerland is not EU but is well-served by German suppliers (CH is in their delivery zone via standard European logistics).

**Assessment:** Good option for a professional GPU server, but pricing is opaque. Expect enterprise-level pricing. Worth requesting a quote for a 2x or 4x GPU configuration. Their focus on rack-mount servers means noise could be an issue for home office use unless you specify quiet operation.

---

### 1.6 RECT (rect.coreto-europe.com) — German, online configurator

RECT manufactures servers and workstations in Friedberg (Hessen), Germany since 2001.

**Relevant Models:**

| Model | Form Factor | GPUs | CPU | Starting Price (EUR) |
|-------|-------------|------|-----|---------------------|
| **WS-8829C** | 4U Tower | Up to 3x RTX PRO 6000 or 2x RTX 5090 | AMD TR PRO 9000WX (96 cores) | EUR 5,195 |
| **WS-8839C** | Rack Workstation | Up to 2x RTX PRO 6000 | AMD EPYC 9005 (160 cores) | EUR 3,515 |
| **WS-8827C5** | 4U Tower | Up to 3 GPUs | AMD TR PRO 5000WX (64 cores) | EUR 3,223 |
| **RS-8639G2** | 2U Rack | Up to 2 dual-slot GPUs | AMD EPYC 9005 | EUR 5,185 |

**Notable features:**
- Online configurator for real-time pricing
- Now featuring AMD Threadripper PRO 9000WX (up to 96 cores, 5.40 GHz)
- AMD EPYC 9005 Turin processors available
- Prices exclude VAT

**Shipping:** German manufacturer, ships within Europe. Contact: +49 6031 6969 21 or service@rect.coreto-europe.com

**Assessment:** The WS-8829C is interesting — it supports up to 3x RTX PRO 6000 Blackwell or 2x RTX 5090 on the latest Threadripper PRO 9000WX platform. Starting at EUR 5,195 (base), a fully configured 2x RTX 5090 system would likely land around EUR 10,000-13,000 (~CHF 9,500-12,400). Worth getting a configured quote through their online tool. Being a German manufacturer also simplifies warranty and logistics for Switzerland.

---

## 2. DIY Build Configurations

### GPU Pricing Reference (Switzerland, February 2026)

| GPU | VRAM | Price per card (CHF) | Source |
|-----|------|---------------------|--------|
| RTX 4090 (24 GB) | 24 GB GDDR6X | ~CHF 1,700-2,900 | toppreise.ch (limited stock, production ceased Oct 2024) |
| RTX 5090 (32 GB) | 32 GB GDDR7 | ~CHF 2,200-2,500 | toppreise.ch (high demand, limited availability) |
| RTX A6000 (48 GB) | 48 GB GDDR6 | ~CHF 4,000-5,200 | toppreise.ch (ASUS version from ~4,046) |
| RTX 6000 Ada (48 GB) | 48 GB GDDR6 | ~CHF 6,063+ | toppreise.ch (PNY version) |
| RTX PRO 6000 Blackwell (96 GB) | 96 GB GDDR7 ECC | ~CHF 6,443 (Workstation Ed.) / ~CHF 6,749 (Server Ed.) | toppreise.ch (NVIDIA direct) |

**RTX PRO 6000 Blackwell — Key addition (February 2026):**
- 96 GB GDDR7 on a single card — the highest VRAM available below CHF 7,000
- 1,792 GB/s bandwidth — same as RTX 5090 but with 3x the VRAM
- 300W TDP — significantly lower than RTX 5090's 575W
- NVLink supported (professional feature)
- Active blower cooler on Workstation Edition (suitable for tower builds)
- With MoE coding models: 134 tok/s on GPT-OSS-120B — far exceeds the 60-80 tok/s target
- This GPU changes the entire recommendation landscape — see updated configs below

**Important notes on RTX 4090:**
- NVIDIA ceased production in October 2024 to transition to RTX 5090
- New stock is increasingly scarce; prices range widely (CHF 1,700 for leftover stock to CHF 2,900+ for remaining new units)
- Consider this a "while stocks last" option

**Important notes on RTX 5090:**
- No NVLink support (consumer card limitation). Multi-GPU relies on PCIe peer-to-peer.
- Ollama does NOT parallelize inference across GPUs — it merely distributes model layers. Don't expect 2x speed from 2 cards.
- 2x RTX 5090 with 64 GB total VRAM can run 70B models at ~27 tokens/s (matching H100 speeds) via model sharding.

**Important notes on RTX A6000:**
- Ampere generation (older than Ada). Still excellent for inference.
- Supports NVLink (2 cards). 4 cards work over PCIe.
- Blower-style cooler = louder than gaming cards in stock form. Consider aftermarket cooling.

---

### 2.0 Config NEW: Single RTX PRO 6000 Blackwell (RECOMMENDED)

| Component | Specification | Est. Price (CHF) | Verified Price (CHF) |
|-----------|--------------|-------------------|----------------------|
| **GPU** | 1x NVIDIA RTX PRO 6000 Blackwell (96 GB) | 6,443 | 6,443 |
| **CPU** | AMD Ryzen 9 9950X (16C/32T) | 550 | 455 |
| **Motherboard** | ASUS TUF Gaming X870E-Plus WIFI7 | 350 | 277 |
| **RAM** | 64 GB DDR5-5600 (2x32GB) Kingston FURY Beast CL36 | 160 | 145 |
| **PSU** | be quiet! Pure Power 12 M 1000W | 180 | 148 |
| **Case** | Fractal Design Define 7 Black | 150 | 130 |
| **CPU Cooler** | Noctua NH-D15 | 100 | 99 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 | 195 |
| **Misc** | Cables | 50 | 50 |
| **TOTAL** | | **~CHF 8,200** | **~CHF 7,950** |

> **Price check: February 16, 2026** — All prices verified against toppreise.ch. System components dropped ~CHF 250 total (mainly CPU and motherboard). GPU price unchanged.

**Shop links (Switzerland):**

| Component | Toppreise (best price) | Digitec | Brack |
|-----------|----------------------|---------|-------|
| RTX PRO 6000 (NVIDIA OEM) | [CHF 6,443](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) | [PNY version CHF 8,744](https://www.digitec.ch/en/s1/product/pny-rtx-pro-6000-blackwell-workstation-edition-96-gb-graphics-card-56565277) | [PNY version](https://www.brack.ch/pny-grafikkarte-nvidia-rtx-pro-6000-blackwell-ws-96-gb-1868148) |
| AMD Ryzen 9 9950X | [CHF 455](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-9-9950X-Granite-Ridge-16x-4-3GHz-100-100001277WOF-p771475) | [digitec](https://www.digitec.ch/de/s1/product/amd-ryzen-9-9950x-am5-430-ghz-16-core-prozessor-47373754) | [brack](https://www.brack.ch/amd-cpu-ryzen-9-9950x-4-3-ghz-1753689) |
| ASUS TUF X870E-Plus WIFI7 | [CHF 277](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-TUF-GAMING-X870E-PLUS-WIFI7-AMD-X870E-90MB1M70-M0EAY0-p809177) | [digitec](https://www.digitec.ch/de/s1/product/asus-tuf-gaming-x870e-plus-wifi7-am5-amd-x870e-atx-mainboard-58291944) | [brack](https://www.brack.ch/fr/asus-carte-mere-tuf-gaming-x870e-plus-wifi7-1891498) |
| Kingston FURY 64GB DDR5-5600 | [CHF 145](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-64GB-KF556C36BBEK2-64-p714466) | [digitec](https://www.digitec.ch/en/s1/product/kingston-fury-beast-2-x-32gb-5600-mhz-ddr5-ram-dimm-ram-20929817) | [brack](https://www.brack.ch/kingston-ddr5-ram-fury-beast-5600-mhz-2x-32-gb-1509447) |
| be quiet! Pure Power 12 M 1000W | [CHF 148](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Pure-Power-12-M-1000-Watts-BN345-p717884) | [digitec](https://www.digitec.ch/en/s1/product/be-quiet-pure-power-12-m-1000-w-pc-netzteil-24077995) | [brack](https://www.brack.ch/be-quiet-netzteil-pure-power-12-m-1000-w-1520808) |
| Samsung 990 Pro 2TB | [CHF 195](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-PRO-NVMe-M-2-SSD-2TB-MZ-V9P2T0BW-p702884) | [digitec](https://www.digitec.ch/en/s1/product/samsung-990-pro-2000-gb-m2-2280-ssd-22097721) | [brack](https://www.brack.ch/samsung-ssd-990-pro-m-2-2280-nvme-2000-gb-1425209) |
| Fractal Design Define 7 | [CHF 130](https://www.toppreise.ch/price-comparison/Computer-cases/FRACTAL-DESIGN-Define-7-Black-FD-C-DEF7A-01-p601634) | [digitec](https://www.digitec.ch/de/s1/product/fractal-define-7-black-solid-atx-matx-mini-itx-e-atx-pc-gehaeuse-12757901) | [brack](https://www.brack.ch/fractal-design-pc-gehaeuse-define-7-schwarz-1288894) |
| Noctua NH-D15 | [CHF 99](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-D15-p366609) | [digitec](https://www.digitec.ch/en/s1/product/noctua-nh-d15-165-mm-cpu-coolers-2580255) | [brack](https://www.brack.ch/noctua-cpu-kuehler-nh-d15-300317) |

> **Note on the GPU:** The cheapest RTX PRO 6000 (CHF 6,443) is the NVIDIA OEM SKU (900-5G144-2200-000) found via toppreise.ch. PNY retail versions at digitec/brack cost CHF 7,800-8,900 — same GPU, just PNY branding and retail box. Also available on [Galaxus (NVIDIA bulk)](https://www.galaxus.ch/en/s1/product/nvidia-rtx-pro-6000-blackwell-works-bulk-96-gb-graphics-card-58324439) and [Interdiscount (PNY)](https://www.interdiscount.ch/de/product/pny-nvidia-rtx-pro-6000-blackwell-fh-dual-96-gb-0014297289).

**VRAM:** 96 GB (single card)
**Pros:** MoE coding models at 134 tok/s far exceed the 60-80 target. 96 GB fits main agent (65 GB) + sub-agent (18 GB) + vision (5 GB) concurrently. Single card means simple AM5 platform — no Threadripper needed. 300W TDP is quiet and runs on standard outlets. NVLink-ready for future second card.
**Cons:** Shared bandwidth under heavy concurrent inference. Dense 70B at ~32 tok/s (meets minimum only).
**Power draw:** ~400W under full GPU load.
**Noise:** Quiet — blower cooler at 300W is far quieter than consumer cards at 450-575W.
**Upgrade path:** Swap to Threadripper platform + add 2nd PRO 6000 for 192 GB.
**Verdict:** **The new #1 recommendation.** MoE models make a single 96 GB card sufficient for all performance targets.

---

### 2.0b Config NEW: RTX PRO 6000 + RTX 5090 (Maximum Capability)

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU 1** | 1x NVIDIA RTX PRO 6000 Blackwell (96 GB) | 6,443 |
| **GPU 2** | 1x NVIDIA RTX 5090 (32 GB) | 2,080 |
| **CPU** | AMD Threadripper 7960X (24C/48T) | 1,025 |
| **Motherboard** | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| **RAM** | 128 GB DDR5-5600 (4x32GB) | 300 |
| **PSU** | be quiet! Dark Power Pro 13 1600W | 370 |
| **Case** | Fractal Design Define 7 XL | 165 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | Cables, fans | 100 |
| **TOTAL** | | **~CHF 11,450** |

**VRAM:** 128 GB (96 + 32)
**Pros:** Zero bandwidth contention — main agent on PRO 6000, sub-agents on RTX 5090. 128 GB meets original VRAM target. Can run largest MoE models (MiniMax-M2.5 at ~130 GB across both GPUs).
**Cons:** More expensive. Requires Threadripper platform. Higher power (875W under load).
**Noise:** Moderate — two GPUs but manageable TDP.
**Verdict:** Worth it if you need dedicated GPUs for main vs sub-agents or want to run the largest MoE models.

---

### 2.1 Config A: 2x RTX 4090 (48 GB total VRAM) — Budget Build

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU** | 2x NVIDIA RTX 4090 24GB | 2x 2,200 = 4,400 |
| **CPU** | AMD Threadripper 7960X (24C/48T) | 1,025 |
| **Motherboard** | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| **RAM** | 128 GB DDR5-5600 (4x32GB non-ECC) | 300 |
| **PSU** | be quiet! Dark Power Pro 13 1600W | 370 |
| **Case** | Fractal Design Define 7 XL | 200 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | Cables, fans, OS (Ubuntu free) | 100 |
| **TOTAL** | | **~CHF 7,365** |

**VRAM:** 48 GB (2x 24 GB)
**Pros:** Lowest cost multi-GPU option. Good for running 70B models at Q4 quantization.
**Cons:** Only 48 GB VRAM is tight. Below the 128 GB minimum target. RTX 4090 stock is dwindling. No NVLink.
**Power draw:** ~1,100W under full GPU load (2x 450W GPU + system)
**Noise:** Moderate — Define 7 XL provides good sound dampening. Gaming GPUs have decent coolers.
**Verdict:** Below minimum VRAM requirement. Only viable if running heavily quantized models (Q3/Q4).

---

### 2.2 Config B: 2x RTX 5090 (64 GB total VRAM) — Current-gen Consumer

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU** | 2x NVIDIA RTX 5090 32GB | 2x 2,350 = 4,700 |
| **CPU** | AMD Threadripper 7960X (24C/48T) | 1,025 |
| **Motherboard** | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| **RAM** | 128 GB DDR5-5600 (4x32GB non-ECC) | 300 |
| **PSU** | be quiet! Dark Power Pro 13 1600W | 370 |
| **Case** | Fractal Design Define 7 XL | 200 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | Cables, fans, OS (Ubuntu free) | 100 |
| **TOTAL** | | **~CHF 7,665** |

**VRAM:** 64 GB (2x 32 GB)
**Pros:** Latest Blackwell architecture. 32 GB per card is a meaningful upgrade over 4090's 24 GB. GDDR7 bandwidth. Excellent single-card performance for models that fit in 32 GB.
**Cons:** Still below 128 GB minimum. No NVLink. RTX 5090 availability is constrained (high demand). Each card draws up to 575W TDP.
**Power draw:** ~1,350W under full GPU load (2x 575W GPU + system). The 1600W PSU should be sufficient but with limited headroom. Consider the ASUS Pro WS 2200W if concerned.
**Noise:** Moderate — RTX 5090 has triple-fan cooler designs. Expect moderate fan noise under full load.
**Benchmark reference:** 2x RTX 5090 running 70B Llama: ~27 tok/s eval rate. Single RTX 5090: ~213 tok/s for smaller models.
**Verdict:** Best price-to-performance for current-gen hardware. Still below 128 GB VRAM target. Good fit if running FP4/Q4 quantized 70B models + smaller sub-agent models.

---

### 2.3 Config C: 4x RTX 4090 (96 GB total VRAM) — Multi-GPU Consumer

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU** | 4x NVIDIA RTX 4090 24GB | 4x 2,200 = 8,800 |
| **CPU** | AMD Threadripper PRO 7965WX (24C/48T) | 2,165 |
| **Motherboard** | ASUS Pro WS WRX90E-SAGE SE (7x PCIe 5.0 x16) | 1,000 |
| **RAM** | 128 GB DDR5-5600 ECC RDIMM (4x32GB) | 500 |
| **PSU** | ASUS Pro WS 3000W Platinum | 950 |
| **Case** | Full tower / open frame (4 triple-slot GPUs require 12+ slots) | 300 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | PCIe risers, extra fans, cables | 200 |
| **TOTAL** | | **~CHF 14,235** |

**VRAM:** 96 GB (4x 24 GB)
**Pros:** Approaches 128 GB VRAM target. Can run 70B FP8 models. WRX90 provides 128 PCIe 5.0 lanes for full bandwidth to all 4 GPUs.
**Cons:** Extreme power draw (~2,000W+ under full load). Significant cooling challenge in a home office. RTX 4090 triple-slot cards in 4x configuration is physically very tight. Stock availability is poor. Very loud under load.
**Power:** 4x 450W (GPU) + 200W (system) = ~2,000W. The 3000W PSU handles this. Requires a dedicated 16A circuit. Standard Swiss outlets are 10A/230V (2,300W max) — may need electrician upgrade.
**Noise:** HIGH. Four air-cooled gaming GPUs under sustained load will be very loud. Custom water cooling loop would add CHF 1,000-2,000.
**Case challenge:** Most tower cases cannot physically fit 4x triple-slot GPUs. Options: open mining frame (ugly but functional), server chassis, or specialty cases like the Thermaltake Core WP200. The Fractal Define 7 XL maxes out at about 3 triple-slot cards.
**Verdict:** At the budget ceiling, physically challenging, very loud, and RTX 4090 stock is scarce. Not recommended unless you specifically need 96 GB with consumer cards.

---

### 2.4 Config D: 2x RTX A6000 (96 GB total VRAM) — Professional

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU** | 2x NVIDIA RTX A6000 48GB | 2x 4,200 = 8,400 |
| **CPU** | AMD Threadripper 7960X (24C/48T) | 1,025 |
| **Motherboard** | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| **RAM** | 128 GB DDR5-5600 (4x32GB non-ECC) | 300 |
| **PSU** | be quiet! Dark Power Pro 13 1600W | 370 |
| **Case** | Fractal Design Define 7 XL | 200 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | NVLink bridge, cables | 150 |
| **TOTAL** | | **~CHF 11,415** |

**VRAM:** 96 GB (2x 48 GB, with NVLink for 2-card interconnect)
**Pros:** 96 GB is close to the 128 GB target. NVLink supported between 2 cards for faster inter-GPU communication. 48 GB per card = can fit large models per GPU. Professional-grade reliability. Dual-slot blower design = more compact than gaming cards.
**Cons:** Ampere generation (2021) — slower per-FLOP than Ada/Blackwell. Blower-style cooler is LOUDER than open-air gaming coolers. A6000 Ampere has lower memory bandwidth than newer cards.
**Power:** ~600W total GPU (2x 300W) + system = ~800W. Very manageable.
**Noise:** The A6000 blower cooler is notoriously loud under sustained load. This is the biggest downside for home office. Aftermarket GPU cooler conversion (e.g., Arctic Accelero, Noctua mod) or adding an AIO water block could mitigate this.
**Verdict:** Excellent VRAM for the money. The main concern is noise from blower coolers. If you can tolerate or mitigate the noise, this is one of the most practical configs for your VRAM needs.

---

### 2.5 Config E: 4x RTX A6000 (192 GB total VRAM) — High-End Professional

| Component | Specification | Est. Price (CHF) |
|-----------|--------------|-------------------|
| **GPU** | 4x NVIDIA RTX A6000 48GB | 4x 4,200 = 16,800 |
| **CPU** | AMD Threadripper PRO 7965WX (24C/48T) | 2,165 |
| **Motherboard** | ASUS Pro WS WRX90E-SAGE SE | 1,000 |
| **RAM** | 128 GB DDR5-5600 ECC RDIMM (4x32GB) | 500 |
| **PSU** | be quiet! Dark Power Pro 13 1600W | 370 |
| **Case** | Full tower (A6000 is dual-slot, 4 cards fit more easily) | 250 |
| **CPU Cooler** | Noctua NH-U14S TR5-SP6 | 120 |
| **Storage** | Samsung 990 Pro 2TB NVMe | 200 |
| **Misc** | NVLink bridges (2x, for 2 pairs), cables | 250 |
| **TOTAL** | | **~CHF 21,655** |

**VRAM:** 192 GB (4x 48 GB)
**Assessment:** Over budget at ~CHF 21,655. The GPUs alone (CHF 16,800) exceed the total budget ceiling. This configuration only becomes viable with:
- Used/refurbished A6000 cards (~CHF 2,000-2,500 each on secondary market, reducing GPU cost to ~CHF 8,000-10,000)
- RTX 6000 Ada would be even more expensive (~CHF 24,000+ for 4 cards)

**With refurbished A6000s (estimated CHF 2,200 each):**

| Component | Est. Price (CHF) |
|-----------|-----------------|
| 4x RTX A6000 (refurbished) | 8,800 |
| Rest of system (same as above) | 4,855 |
| **TOTAL** | **~CHF 13,655** |

This brings it within budget, but with the risk caveats around refurbished GPU purchases.

**Power:** 4x 300W (GPU) + 200W (system) = ~1,400W. The 1600W PSU is adequate.
**Noise:** Four blower-style A6000 cards will be VERY LOUD. Aftermarket cooling is essentially mandatory for home office.
**Physical fit:** A6000 is dual-slot (not triple-slot like 4090), so 4 cards fit in standard motherboards with proper spacing. The WRX90E-SAGE SE has 7 PCIe x16 slots — physical clearance is feasible.

---

## 3. Base System Component Analysis

### 3.1 CPU Platform

For multi-GPU AI inference, the CPU is not the bottleneck — the GPUs do the heavy lifting. What matters is **PCIe lane count** for GPU bandwidth.

| Platform | PCIe Lanes | Max GPUs at x16 | Socket | RAM Type | Notes |
|----------|-----------|-----------------|--------|----------|-------|
| **AMD Threadripper 7000** (TRX50) | 48 PCIe 5.0 | 3 (x16 each) | sTR5 | DDR5 (non-ECC) | Consumer HEDT. Good for 2-3 GPUs. |
| **AMD Threadripper PRO 7000WX** (WRX90) | 128 PCIe 5.0 | 7 (x16 each) | sTR5 | DDR5 ECC RDIMM | Professional. Required for 4+ GPUs. |
| **Intel Xeon W-2500/3500** | Up to 112 PCIe 5.0 | 4-5 (x16 each) | LGA 4677 | DDR5 ECC | BIZON G3000 uses this. |
| **AMD EPYC 9005 (Turin)** | 128 PCIe 5.0 | 7+ | SP5 | DDR5 ECC RDIMM | Server-class. Overkill for inference. |

**Recommendation:**
- **2 GPUs:** AMD Threadripper 7960X (24C, CHF 1,025) on TRX50. Best value.
- **3-4 GPUs:** AMD Threadripper PRO 7965WX (24C, CHF 2,165) on WRX90. Necessary for lane count.
- You do NOT need a high core count CPU for inference. 24 cores is more than enough. Save money here.

### 3.2 Motherboard

| Motherboard | Chipset | PCIe x16 Slots | RAM Slots | Price (CHF) |
|------------|---------|----------------|-----------|-------------|
| **ASUS Pro WS TRX50-SAGE WIFI** | TRX50 | 4 (3x PCIe 5.0, 1x PCIe 4.0) | 4x DDR5 | ~650 |
| **ASUS Pro WS TRX50-SAGE WIFI A** | TRX50 | 4 | 4x DDR5 | ~641 |
| **ASUS Pro WS WRX90E-SAGE SE** | WRX90 | 7x PCIe 5.0 x16 | 8x DDR5 | ~1,000 |

- TRX50 boards are for 2-3 GPU builds (consumer Threadripper)
- WRX90 boards are for 4+ GPU builds (Threadripper PRO)
- Both support CEB/EEB form factors (larger than standard ATX)

### 3.3 RAM

For inference workloads, system RAM speed barely matters. The models live in VRAM. System RAM just needs to be sufficient for the OS, model loading, and serving framework.

| Option | Capacity | Type | Price (CHF) |
|--------|----------|------|-------------|
| Kingston FURY Beast DDR5-5600 128GB (4x32GB) | 128 GB | Non-ECC UDIMM | ~300 |
| Kingston FURY Renegade Pro DDR5-5600 128GB (4x32GB) | 128 GB | ECC RDIMM | ~500-700 |

- Non-ECC is fine for home use. ECC adds cost but better reliability for 24/7 operation.
- 64 GB minimum, 128 GB recommended (for loading/preprocessing large models before GPU transfer).

### 3.4 Power Supply

| PSU | Wattage | Efficiency | Price (CHF) | Best For |
|-----|---------|-----------|-------------|----------|
| **be quiet! Dark Power Pro 13** | 1600W | 80+ Titanium | ~370 | 2 GPU builds |
| **ASUS Pro WS 2200W Platinum** | 2200W | 80+ Platinum | ~600-700 (est.) | 3 GPU builds |
| **ASUS Pro WS 3000W Platinum** | 3000W | 80+ Platinum | ~950 (EUR 899) | 4 GPU builds |

**Wattage guidelines:**
- 2x RTX 4090: 1,200W minimum, 1,600W recommended
- 2x RTX 5090: 1,350W minimum, 1,600W recommended (tight)
- 4x RTX 4090: 2,200W minimum, 3,000W recommended
- 4x RTX A6000: 1,400W minimum, 1,600W sufficient
- 2x RTX A6000: 800W minimum, 1,000W sufficient

**Swiss electrical note:** Standard Swiss residential circuits are 10A/230V = 2,300W max per outlet. A 4x RTX 4090 build drawing 2,000W+ will need a dedicated circuit. 2-GPU builds are fine on standard outlets.

### 3.5 Case

| Case | Type | Max GPU Length | Fits 4 GPUs? | Price (CHF) | Noise |
|------|------|---------------|--------------|-------------|-------|
| **Fractal Design Define 7 XL** | Full Tower | 491mm | 3 triple-slot max | ~163-200 | Very quiet |
| **be quiet! Dark Base Pro 901** | Full Tower | 490mm | 3 triple-slot max | ~280-320 | Very quiet |
| **Thermaltake Core WP200** | Super Tower | 400mm+ | Yes (modular) | ~250-350 | Moderate |
| **Open mining frame** | Open frame | Unlimited | Yes | ~50-100 | N/A (open) |

**Key insight:** Fitting 4x triple-slot GPUs (like RTX 4090/5090) in a standard case is extremely difficult. Most full towers only have 7-8 expansion slots, and triple-slot cards need 3 slots each (= 12 slots for 4 cards).

Solutions for 4 GPUs:
- Use dual-slot professional cards (A6000 = dual-slot, so 4 cards fit in 8 slots)
- Use a 4U rack chassis (designed for multi-GPU)
- Use an open-air mining frame (functional but not quiet)
- Use the ASUS Pro WS TRX50 or WRX90 boards which have designed slot spacing for multi-GPU

**For home office silence: Fractal Define 7 XL or be quiet! Dark Base Pro 901 for 2-GPU builds.**

### 3.6 CPU Cooling

| Cooler | Socket | Noise | Price (CHF) |
|--------|--------|-------|-------------|
| **Noctua NH-U14S TR5-SP6** | sTR5 (Threadripper) | Very quiet | ~120 |
| **Noctua NH-D9 TR5-SP6 4U** | sTR5 (Threadripper) | Quiet | ~120 |

- Air cooling is preferred for reliability and lower maintenance in a 24/7 headless server
- Threadripper 7960X TDP is 350W but actual inference load is minimal (CPU barely used during inference)
- Noctua's TR5 coolers are purpose-built for the large sTR5 socket

### 3.7 Storage

| Drive | Capacity | Speed | Price (CHF) |
|-------|----------|-------|-------------|
| Samsung 990 Pro 2TB | 2 TB | 7,450 MB/s read | ~195-204 |
| Samsung 990 Pro 4TB | 4 TB | 7,450 MB/s read | ~337 |
| Samsung 990 EVO Plus 2TB | 2 TB | 5,000 MB/s read | ~114 |

- Fast NVMe matters for model loading times (loading a 70B model into VRAM)
- 2 TB is sufficient for many models. 4 TB gives breathing room for multiple model versions.
- A 70B model at FP8 is ~70 GB. At Q4, ~35 GB. Loading from NVMe takes seconds.

### 3.8 Air vs. Liquid Cooling for Noise

**Air Cooling (recommended for 2-GPU builds):**
- Simpler, more reliable, zero maintenance
- Noctua fans + Fractal sound-dampened case = very quiet idle, moderate under load
- Consumer GPUs (4090/5090) have good air coolers already
- Total additional cost: CHF 0 (fans included with case and cooler)

**AIO Liquid Cooling (GPU):**
- Companies like EKWB and Alphacool offer GPU water blocks
- Significantly reduces GPU noise
- Adds complexity, cost, and failure points
- Estimated additional cost: CHF 300-500 per GPU for AIO
- Recommended for professional blower-style cards (A6000) in home office

**Custom Loop Liquid Cooling:**
- Best noise reduction possible
- CPU + all GPUs on one loop
- Very expensive: CHF 1,000-2,500 for a full loop
- Maintenance required (fluid changes every 1-2 years)
- Only recommended if noise is absolutely critical AND you're running 3+ GPUs

---

## 4. Configuration Cost Summaries

### All prices in CHF, estimated February 2026

| Config | GPUs | Total VRAM | Est. Total Cost | Budget Fit? | VRAM Target? | Noise | Power Draw |
|--------|------|-----------|----------------|-------------|-------------|-------|-----------|
| **NEW: 1x RTX PRO 6000** | 1x 96GB | 96 GB | **~CHF 8,200** | Yes (sweet spot) | Meets (96GB) | Quiet | ~400W |
| **NEW: PRO 6000 + RTX 5090** | 96+32GB | 128 GB | **~CHF 11,450** | Yes | Meets target | Moderate | ~875W |
| **A: 2x RTX 4090** | 2x 24GB | 48 GB | **~CHF 7,365** | Yes (mid-range) | Below minimum | Moderate | ~1,100W |
| **B: 2x RTX 5090** | 2x 32GB | 64 GB | **~CHF 7,665** | Yes (mid-range) | Below minimum | Moderate | ~1,350W |
| **C: 4x RTX 4090** | 4x 24GB | 96 GB | **~CHF 14,235** | Barely | Near minimum | LOUD | ~2,000W |
| **D: 2x RTX A6000** | 2x 48GB | 96 GB | **~CHF 11,415** | Yes | Near minimum | Loud (blower) | ~800W |
| **E: 4x RTX A6000 (new)** | 4x 48GB | 192 GB | **~CHF 21,655** | Over budget | Exceeds target | VERY LOUD | ~1,400W |
| **E': 4x RTX A6000 (refurb)** | 4x 48GB | 192 GB | **~CHF 13,655** | Yes (top end) | Exceeds target | VERY LOUD | ~1,400W |

### Pre-Built Comparison

| Vendor | Config | VRAM | Est. Total (CHF, incl. shipping/import) | Notes |
|--------|--------|------|----------------------------------------|-------|
| **BIZON G3000 + 2x RTX 5090** | 2x 32GB | 64 GB | ~CHF 14,000-16,500 | Turnkey, water-cooled option available. At/over budget limit. |
| **BIZON X5500 G2 + 2x RTX 5090** | 2x 32GB | 64 GB | ~CHF 13,000-15,000 | AMD Threadripper PRO base. Turnkey. |
| **RECT WS-8829C + 2x RTX 5090** | 2x 32GB | 64 GB | ~CHF 10,000-13,000 (est.) | German, easier warranty for CH. Use configurator for exact price. |
| **BRESSNER Custom** | Varies | Varies | Request quote | German, enterprise focus. |

---

## 5. Recommendations

### Updated Recommendation (February 2026 — MoE Models + RTX PRO 6000)

The discovery of MoE coding models and the availability of the RTX PRO 6000 Blackwell fundamentally changes the recommendation landscape. The previous analysis focused on dense 70B models requiring massive multi-GPU setups. With MoE models achieving 134 tok/s on a single PRO 6000, most of those configurations are now unnecessary.

#### Option 1: "Best Overall" — Single RTX PRO 6000 DIY (~CHF 8,200)
- **96 GB VRAM** — sufficient for MoE main agent + sub-agents + vision concurrently
- **134 tok/s** on GPT-OSS-120B (MoE) — far exceeds 60-80 target
- Simple AM5 platform — no Threadripper needed
- Quiet, low power (400W), fits standard outlets
- **Best for:** Getting the best performance per CHF with room for future upgrades

#### Option 2: "Zero Contention" — RTX PRO 6000 + RTX 5090 (~CHF 11,450)
- **128 GB VRAM** — dedicated GPUs for main vs sub-agents
- No bandwidth sharing between workloads
- Can run largest MoE models (MiniMax-M2.5 at ~130 GB across both GPUs)
- **Best for:** Heavy concurrent workloads or future-proofing with 128 GB

#### Option 3: "Turnkey Professional" — RECT WS-8829C configured (~CHF 10,000-13,000 est.)
- German manufacturer, straightforward warranty for Switzerland
- Now supports up to 3x RTX PRO 6000 Blackwell
- Threadripper PRO 9000WX platform
- Professionally assembled and tested
- **Best for:** Those who prefer not to build and want European warranty support

#### Option 4: "Budget NVIDIA" — 2x RTX 5090 DIY (~CHF 7,090)
- **64 GB VRAM** — below 96 GB practical minimum
- Runs MoE models fast but tight on concurrent model loading
- **Best for:** Tightest budget with NVIDIA preference

### What I Would NOT Recommend (Updated)

- **4x RTX 4090 / 4x RTX A6000 / 4x RTX 3090:** Multi-GPU dense model setups are now superseded by single PRO 6000 + MoE models. The single card is faster, quieter, cheaper, and simpler.
- **2x RTX 4090:** Only 48 GB VRAM, stock scarce, overtaken by PRO 6000
- **2x RTX A6000 (new):** CHF 11,415 for 96 GB — same VRAM as a single PRO 6000 at CHF 8,200 but slower (older Ampere architecture)
- **BIZON pre-built:** Good products but price premium + import duties pushes it over budget for equivalent specs
- **Puget Systems / Lambda Labs / Joule Performance:** Not available or not suitable (see vendor details above)

### Key Decision Factors (Updated)

1. **MoE vs Dense:** MoE coding models (GPT-OSS-120B, Qwen3-Coder-30B-A3B) are now the primary workload. Dense 70B models are fallbacks.
2. **Single GPU vs Multi-GPU:** A single RTX PRO 6000 (96 GB) with MoE models outperforms multi-GPU dense setups at lower cost
3. **NVIDIA vs Apple:** PRO 6000 (~CHF 8,200): 134 tok/s MoE, CUDA, upgradeable. Mac Studio (~CHF 6,100): 69 tok/s MoE, silent, 256 GB memory
4. **Build vs Buy:** DIY PRO 6000 build saves CHF 2,000-5,000 over pre-built. RECT (German) is a good pre-built option.
5. **Noise:** PRO 6000 at 300W is quiet. Previous 4-GPU configs were loud.

---

## Sources

- [BIZON G3000 Product Page](https://bizon-tech.com/bizon-g3000.html)
- [BIZON G7000 Product Page](https://bizon-tech.com/bizon-g7000.html)
- [BIZON Shipping Policy](https://bizon-tech.com/shipping)
- [BIZON Warranty](https://bizon-tech.com/warranty)
- [BIZON X5500 G2](https://bizon-tech.com/bizon-x5500.html)
- [Joule Performance Workstations](https://www.jouleperformance.com/ch_en/professional-category/workstation)
- [Lambda Labs Legacy Hardware](https://lambda.ai/legacy-hardware)
- [Puget Systems International Policy](https://www.pugetsystems.com/international-policies/)
- [Puget Systems Multi-GPU Workstation](https://www.pugetsystems.com/solutions/ai-and-hpc-workstations/generative-ai/multi-gpu-rackmount-workstation/)
- [BRESSNER GPU Servers](https://shop.bressner.de/en/products/hpc-solutions/gpu-computers/gpu-servers/)
- [RECT GPU Servers](https://www.rect.coreto-europe.com/en/gpu-servers/gpu-servers.html)
- [ASUS Pro WS TRX50-SAGE WIFI — CHF 648.95](https://www.toppreise.ch/price-comparison/Motherboards/ASUS-Pro-WS-TRX50-SAGE-WIFI-AMD-TRX50-90MB1FZ0-M0EAY0-p756606)
- [ASUS Pro WS WRX90E-SAGE SE — CHF 995.90](https://webshop.asus.com/ch-en/90MB1FW0-M0EAY0/Pro-WS-WRX90E-SAGE-SE)
- [AMD Threadripper 7960X — CHF 1,023.20](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-7960X-Storm-Peak-24x-4-2GHz-100-000001352-p751329)
- [AMD Threadripper PRO 7965WX — CHF 2,161.05](https://www.toppreise.ch/price-comparison/Processors/AMD-Ryzen-Threadripper-PRO-7965WX-Storm-100-100000885WOF-p751270)
- [RTX 5090 from CHF 2,224 (Palit)](https://www.toppreise.ch/productcollection/GeForce_RTX_5090-pc-s82250)
- [RTX A6000 (ASUS) from CHF 4,046](https://www.toppreise.ch/price-comparison/Graphics-cards/ASUS-Nvidia-RTX-A6000-Nvidia-RTX-A6000-48GB-GDDR6-90SKC000-M5EAN0-p716143)
- [RTX 6000 Ada from CHF 6,063](https://www.toppreise.ch/price-comparison/Graphics-cards/PNY-Nvidia-RTX-6000-Ada-Generation-Nvidia-VCNRTX6000ADA-PB-p716156)
- [Fractal Design Define 7 XL from CHF 162.90](https://www.toppreise.ch/productcollection/Define_7_XL-pc-s47616)
- [be quiet! Dark Power Pro 13 1600W — CHF 368.25](https://www.toppreise.ch/price-comparison/Power-supplies/BE-QUIET-Dark-Power-Pro-13-1600-Watts-BN332-p729058)
- [Noctua NH-U14S TR5-SP6 — CHF 119.85](https://www.toppreise.ch/price-comparison/CPU-coolers/NOCTUA-NH-U14S-TR5-SP6-p750500)
- [Samsung 990 Pro 2TB — CHF 194.95-203.98](https://www.toppreise.ch/price-comparison/Solid-State-Drives-SSD/SAMSUNG-990-PRO-NVMe-M-2-SSD-2TB-MZ-V9P2T0BW-p702884)
- [Kingston FURY Beast DDR5-5600 128GB — CHF 300.45](https://www.toppreise.ch/price-comparison/Memory/KINGSTON-FURY-Beast-Kit-DDR5-5600-128GB-KF556C40BBK4-128-p734307)
- [ASUS Pro WS 3000W Platinum PSU](https://www.asus.com/motherboards-components/power-supply-units/workstation/asus-pro-ws-3000p/)
- [Dual RTX 5090 Ollama Benchmark (DatabaseMart)](https://www.databasemart.com/blog/ollama-gpu-benchmark-rtx5090-2)
- [RTX 5090 LLM Benchmarks (Hardware Corner)](https://www.hardware-corner.net/rtx-5090-llm-benchmarks/)
- [RTX 4090 vs 5090 for AI](https://www.bestgpusforai.com/gpu-comparison/5090-vs-4090)
- [ASUS Pro WS 3000W PSU Launch (Guru3D)](https://www.guru3d.com/story/asus-launches-pro-ws-platinum-3000w-psu-for-multigpu-workstations/)
- [Puget Systems Dual RTX 5090 Approach](https://www.pugetsystems.com/blog/2025/10/28/our-approach-to-dual-geforce-rtx-5090-workstations/)
- [RTX 4090 Switzerland from CHF 1,652.80 (MSI)](https://www.toppreise.ch/productcollection/GeForce_RTX_4090-pc-s63606)
