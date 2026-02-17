# Alternative Hardware Platforms for Local AI Inference (2025-2026)

> Research date: February 2026. Comprehensive survey of all hardware platforms competing with NVIDIA DGX Spark, NVIDIA RTX PRO 6000, and Apple Silicon (Mac Studio/Mac Pro) for local AI inference.

---

## Table of Contents

1. [AMD Alternatives](#1-amd-alternatives)
2. [Intel Alternatives](#2-intel-alternatives)
3. [Qualcomm](#3-qualcomm)
4. [Cerebras](#4-cerebras)
5. [Groq](#5-groq)
6. [SambaNova](#6-sambanova)
7. [Tenstorrent](#7-tenstorrent)
8. [Google TPU](#8-google-tpu)
9. [Amazon Trainium / Inferentia](#9-amazon-trainium--inferentia)
10. [Chinese Alternatives](#10-chinese-alternatives)
11. [AI Chip Startups](#11-ai-chip-startups)
12. [FPGA-Based Solutions](#12-fpga-based-solutions)
13. [Multi-GPU Consumer Setups](#13-multi-gpu-consumer-setups)
14. [Purpose-Built Inference Appliances](#14-purpose-built-inference-appliances)
15. [Photonic / Emerging Technologies](#15-photonic--emerging-technologies)
16. [Summary Comparison Table](#16-summary-comparison-table)
17. [Relevance Assessment for Our Use Case](#17-relevance-assessment-for-our-use-case)

---

## 1. AMD Alternatives

### AMD Instinct MI300X (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 192 GB HBM3E |
| **Memory Bandwidth** | 5,300 GB/s |
| **Compute (FP16)** | 1,307 TFLOPS |
| **TDP** | 750W |
| **Price** | ~$10,000-15,000/card (not sold standalone retail; cloud: ~$1.49-2.20/hr) |
| **Availability** | Available via cloud providers (Vultr, TensorWave, DigitalOcean). Not sold as individual retail cards. |
| **Key Differentiator** | 192 GB HBM3E on a single card — more memory than any NVIDIA competitor at the time of launch. |

**Assessment for our use case:** Not purchasable as a discrete card for a home workstation. Data center class with passive cooling requiring server chassis airflow. Cloud-only access model. **Not viable.**

Sources: [AMD MI300X](https://www.amd.com/en/products/accelerators/instinct/mi300.html), [MI300X Pricing](https://getdeploying.com/gpus/amd-mi300x), [BentoML AMD GPU Guide](https://www.bentoml.com/blog/amd-data-center-gpus-mi250x-mi300x-mi350x-and-beyond)

---

### AMD Instinct MI325X (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 256 GB HBM3E |
| **Memory Bandwidth** | 6,000 GB/s |
| **Compute** | Improved over MI300X |
| **TDP** | ~750W |
| **Price** | Not publicly priced; data center sales only |
| **Availability** | Shipping since Q4 2024 |
| **Key Differentiator** | 256 GB HBM3E — highest single-card memory of any GPU in 2024-2025. |

**Assessment:** Same constraints as MI300X. Data center only. **Not viable.**

Source: [AMD Instinct Roadmap](https://ir.amd.com/news-events/press-releases/detail/1201/amd-accelerates-pace-of-data-center-ai-innovation-and-leadership-with-expanded-amd-instinct-gpu-roadmap)

---

### AMD Instinct MI350X / MI355X (Data Center, June 2025)

| Spec | Value |
|------|-------|
| **Memory** | 288 GB HBM3E |
| **Memory Bandwidth** | ~8,000 GB/s (estimated) |
| **Compute** | Up to 4x peak theoretical over MI300X |
| **TDP** | 1,000W (MI350X, air) / 1,400W (MI355X, liquid) |
| **Price** | Not publicly priced |
| **Availability** | Launched June 2025 |
| **Key Differentiator** | CDNA 4 architecture. 4x performance leap over MI300X. |

**Assessment:** Power-hungry data center accelerators. 1,000-1,400W per card is not feasible for home use. **Not viable.**

Sources: [AMD MI350 Series](https://www.amd.com/en/products/accelerators/instinct/mi350.html), [AMD MI350 Blog](https://www.amd.com/en/blogs/2025/amd-instinct-mi350-series-game-changer.html)

---

### AMD Radeon PRO W7900 (Workstation)

| Spec | Value |
|------|-------|
| **Memory** | 48 GB GDDR6 ECC |
| **Memory Bandwidth** | 864 GB/s |
| **Compute (FP32)** | 61.3 TFLOPS |
| **TDP** | 295W |
| **Price** | ~$3,999 USD / ~CHF 4,200 |
| **Availability** | Available now |
| **Key Differentiator** | 48 GB ECC memory at a competitive price. RDNA 3 with AI accelerators. Half the cost of NVIDIA RTX 6000 Ada. |

**Assessment:** Decent VRAM at a reasonable price, but memory bandwidth (864 GB/s) is less than half the RTX PRO 6000 Blackwell (1,792 GB/s). For inference, bandwidth is king. ROCm software ecosystem is still significantly behind CUDA — many inference engines (vLLM, Ollama, llama.cpp) have AMD support but with lower performance and more bugs. For a 70B Q4 model (~36 GB): ~24 tok/s estimated vs ~32 tok/s on RTX PRO 6000. **Marginal — software ecosystem risk is the dealbreaker.**

Sources: [AMD W7900](https://www.amd.com/en/products/graphics/workstations/radeon-pro/w7900.html), [W7900 Review](https://aecmag.com/workstations/review-amd-radeon-pro-w7900-and-pro-w7800/)

---

### AMD Radeon PRO W7800 (Workstation)

| Spec | Value |
|------|-------|
| **Memory** | 32 GB GDDR6 |
| **Memory Bandwidth** | ~576 GB/s |
| **Compute (FP32)** | 45 TFLOPS |
| **TDP** | 260W |
| **Price** | ~$2,499 USD |
| **Availability** | Available now |
| **Key Differentiator** | Budget workstation option with ECC memory. |

**Assessment:** Only 32 GB VRAM and low bandwidth. **Not competitive for our use case.**

Source: [W7800 Review](https://develop3d.com/workstations/review-amd-radeon-pro-w7900-and-pro-w7800/)

---

### AMD Strix Halo (Ryzen AI MAX+ 395) — Desktop APU

| Spec | Value |
|------|-------|
| **Memory** | Up to 128 GB LPDDR5X-8000 (unified, up to 96 GB allocatable as VRAM) |
| **Memory Bandwidth** | 256 GB/s |
| **Compute** | Radeon 8060S iGPU (40 CUs), ~RTX 4060-4070 laptop class |
| **CPU** | 16 Zen 5 cores, 32 threads, 5.1 GHz max |
| **TDP** | 45-125W |
| **Price** | ~$1,500-2,000 (mini-PC with 128 GB); ~$3,300-3,400 (PRO workstation with 64 GB) |
| **Availability** | Available now in mini-PCs (Beelink GTR9 Pro, Framework Desktop, GMKtec EVO-X2, X+ XRIVAL) |
| **Key Differentiator** | DGX Spark competitor. x86 CPU with unified memory. Runs Windows and Linux. Under $2,000 for 128 GB. |

**Assessment:** Direct DGX Spark competitor at potentially lower price. However, 256 GB/s memory bandwidth is very low — roughly 7x slower than RTX PRO 6000. Estimated inference: ~7 tok/s on 70B Q4 dense, ~26 tok/s on MoE with 10 GB active weights. ROCm support for the integrated GPU is immature. Interesting for budget experimentation but **too slow for production coding inference.**

Sources: [AMD Strix Halo vs DGX Spark](https://www.theregister.com/2025/12/25/amd_strix_halo_nvidia_spark/), [Framework Desktop Review](https://www.servethehome.com/framework-desktop-review-a-solid-amd-strix-halo/), [Beelink GTR9 Pro Review](https://www.servethehome.com/beelink-gtr9-pro-review-amd-ryzen-ai-max-395-system-with-128gb-and-dual-10gbe/)

---

## 2. Intel Alternatives

### Intel Gaudi 3 (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 128 GB HBM2E |
| **Memory Bandwidth** | 3,670 GB/s |
| **Compute** | 1,835 BF16/FP8 TFLOPS |
| **TDP** | ~600W |
| **Price** | ~$15,625/chip (~$125,000 for 8-chip baseboard) |
| **Availability** | Available through Dell, IBM Cloud, and OEM partners |
| **Key Differentiator** | 70% better price-performance inference throughput on Llama 3 80B vs NVIDIA H100. 24x 200GbE networking integrated. |

**Assessment:** Enterprise/data center only. Requires OEM server chassis. Not sold as individual PCIe cards. The 8-chip baseboard at $125K is well beyond budget. **Not viable for home use.**

Sources: [Intel Gaudi 3](https://www.intel.com/content/www/us/en/products/details/processors/ai-accelerators/gaudi.html), [Tom's Hardware Gaudi 3](https://www.tomshardware.com/tech-industry/artificial-intelligence/intel-launches-gaudi-3-accelerator-for-ai-slower-than-h100-but-also-cheaper), [Dell Gaudi 3](https://www.dell.com/en-us/lp/dt/intel-gaudi)

---

### Intel Arc Pro B60 (Workstation)

| Spec | Value |
|------|-------|
| **Memory** | 24 GB GDDR6 |
| **Memory Bandwidth** | 456 GB/s |
| **Compute (INT8)** | Not disclosed; competitive with mid-range |
| **TDP** | Not disclosed |
| **Price** | ~$5,000-10,000 for 8-GPU / 192 GB VRAM configuration |
| **Availability** | Sampling Q3 2025, broad availability Q4 2025 |
| **Key Differentiator** | Outperforms NVIDIA L40S by up to 4x in MLPerf v5.1 (Llama 8B). Multi-GPU scaling. |

**Assessment:** Interesting value proposition for multi-GPU setups. 8x GPUs = 192 GB at competitive pricing. But: immature software ecosystem (Intel's OpenVINO/oneAPI vs CUDA), limited real-world inference benchmarks for large models, and Q4 2025 availability means limited track record. **Worth monitoring but too risky for primary investment now.**

Sources: [Intel Arc Pro B60 vLLM](https://blog.vllm.ai/2025/11/11/intel-arc-pro-b.html), [Computex 2025 Arc Pro B-Series](https://www.igorslab.de/en/computex-2025-intel-arc-pro-b-series-new-gpus-for-workstation-and-ki-inference-at-a-glance/)

---

### Intel Arc Pro B50 (Workstation, Entry Level)

| Spec | Value |
|------|-------|
| **Memory** | 16 GB GDDR6 |
| **Memory Bandwidth** | 224 GB/s |
| **Compute (INT8)** | 170 TOPS |
| **TDP** | 70W (slot-powered) |
| **Price** | $349 MSRP |
| **Availability** | Available now |
| **Key Differentiator** | Cheapest professional GPU with 16 GB for AI inference at 70W. |

**Assessment:** Only 16 GB VRAM. Too small for anything beyond 7-13B models. **Not sufficient for our use case.**

Source: [Intel Arc Pro B50 Review](https://www.pugetsystems.com/labs/articles/intel-arc-pro-b50-review/)

---

### Intel Crescent Island (Upcoming, Xe3P)

| Spec | Value |
|------|-------|
| **Memory** | 160 GB LPDDR5X |
| **Memory Bandwidth** | Not disclosed (LPDDR5X = likely ~500-700 GB/s) |
| **Compute** | Optimized for inference, supports INT8/FP8/FP16/BF16 |
| **TDP** | Air-cooled, "cost-optimized" |
| **Price** | Not disclosed |
| **Availability** | Sampling H2 2026, likely shipping 2027 |
| **Key Differentiator** | First inference-only data center GPU from Intel. 160 GB memory. Xe3P architecture. |

**Assessment:** Very promising specs — 160 GB in an air-cooled, cost-optimized form factor could be a game-changer for local inference. But sampling in H2 2026 means it won't be available for at least a year. **Future watch item, not actionable now.**

Sources: [Tom's Hardware Crescent Island](https://www.tomshardware.com/pc-components/gpus/intel-unveils-crescent-island-an-inference-only-gpu-with-xe3p-architecture-and-160gb-of-memory), [Phoronix Crescent Island](https://www.phoronix.com/review/intel-crescent-island)

---

## 3. Qualcomm

### Qualcomm Cloud AI 100 Ultra (Data Center Inference)

| Spec | Value |
|------|-------|
| **Memory** | 128 GB on-card DRAM + 576 MB on-die SRAM |
| **Memory Bandwidth** | 548 GB/s (DRAM) |
| **Compute (INT8)** | 870 TOPS |
| **TDP** | Not disclosed |
| **Price** | Not publicly disclosed |
| **Availability** | Available through partners (Gigabyte, Cirrascale) |
| **Key Differentiator** | 4x AI 100 SoCs on a single PCIe card. Purpose-built for inference. |

**Assessment:** Decent memory capacity (128 GB) but low bandwidth (548 GB/s). Software ecosystem is highly proprietary (Qualcomm AI Engine). Very limited model support compared to CUDA. **Not practical for general-purpose LLM inference in a home setup.**

Sources: [Qualcomm Cloud AI 100 Ultra](https://www.qualcomm.com/artificial-intelligence/data-center/cloud-ai-100-ultra), [Qualcomm Cloud AI 100 Ultra Product Brief](https://www.qualcomm.com/content/dam/qcomm-martech/dm-assets/documents/Prod-Brief-QCOM-Cloud-AI-100-Ultra.pdf)

---

### Qualcomm Dragonwing AI On-Prem Appliance (Enterprise)

| Spec | Value |
|------|-------|
| **Form Factor** | Desktop or wall-mounted appliance |
| **Use Case** | Enterprise edge AI inference (chatbots, RAG, transcription, image gen) |
| **Software** | Qualcomm AI Inference Suite with OpenAI-compatible APIs |
| **Price** | Not disclosed |
| **Availability** | Announced CES 2025, available through Aetina and other partners |
| **Key Differentiator** | Turnkey on-prem AI box with enterprise software suite. Low power. |

**Assessment:** Interesting concept for enterprise deployment but not designed for high-performance LLM inference at frontier model scale. Edge/enterprise focus. **Not suitable for coding-grade inference.**

Sources: [Qualcomm On-Prem AI Appliance](https://www.qualcomm.com/news/releases/2025/01/qualcomm-launches-on-prem-ai-appliance-solution-and-inference-su), [Qualcomm Dragonwing Blog](https://www.qualcomm.com/developer/blog/2025/03/qualcomm-ai-on-prem-appliance-and-inference-suite)

---

### Qualcomm Snapdragon X2 Elite (AI PC Processor)

| Spec | Value |
|------|-------|
| **Memory** | Up to 128 GB (system RAM, shared) |
| **Memory Bandwidth** | Up to 228 GB/s (X2 Elite Extreme) |
| **NPU Performance** | 80 TOPS (INT8) |
| **CPU** | 18 cores (12 Prime + 6 Performance) |
| **TDP** | Laptop-class (~45-65W) |
| **Price** | Laptop pricing (~$1,500-3,000 for equipped systems) |
| **Availability** | H1 2026 |
| **Key Differentiator** | ARM-based AI PC with 80 TOPS NPU. Excellent power efficiency. |

**Assessment:** NPU is designed for small on-device models (7-13B), not frontier LLM inference. 228 GB/s bandwidth is far too low for competitive inference speeds. **Not viable for our workload.**

Sources: [Qualcomm Snapdragon X2 Elite](https://www.qualcomm.com/news/releases/2025/09/new-snapdragon-x2-elite-extreme-and-snapdragon-x2-elite-are-the-), [Futurum Snapdragon X2](https://futurumgroup.com/insights/snapdragon-x2-elite-pushes-ai-pc-performance-to-new-heights/)

---

## 4. Cerebras

### Cerebras CS-3 (WSE-3 Wafer-Scale Engine)

| Spec | Value |
|------|-------|
| **Chip Size** | 46,225 mm^2 (full wafer, 21.5cm per side) |
| **Cores** | 900,000 AI-optimized cores |
| **On-Chip SRAM** | 44 GB |
| **Memory Bandwidth** | 7,000x more than NVIDIA H100 (on-chip) |
| **Process** | 5nm |
| **Transistors** | 4 trillion |
| **Price** | Enterprise sales only, estimated millions per system |
| **Availability** | Available for enterprise/on-premises deployment |
| **Key Differentiator** | Entire model fits in on-chip SRAM — no memory wall. World's fastest inference per chip. |

**Assessment:** The most radical architecture in AI computing — an entire wafer as a single chip. Spectacular performance but completely inaccessible for individual/SOHO use. Requires dedicated infrastructure, cooling, and multi-million dollar budgets. Cerebras is targeting IPO in Q2 2026. **Fascinating technology, entirely out of scope.**

Sources: [Cerebras Chip](https://www.cerebras.ai/chip), [Cerebras Inference Launch](https://www.cerebras.ai/press-release/cerebras-launches-the-worlds-fastest-ai-inference), [IEEE Spectrum WSE-3](https://spectrum.ieee.org/cerebras-chip-cs3)

---

## 5. Groq

### Groq LPU (Language Processing Unit)

| Spec | Value |
|------|-------|
| **Memory** | 230 MB SRAM per chip (no HBM/DRAM) |
| **Memory Bandwidth** | Up to 80 TB/s on-die |
| **Compute (INT8)** | 750 TOPS |
| **Compute (FP16)** | 188 TFLOPS |
| **TDP** | Not disclosed |
| **Price** | ~$19,948 per GroqCard Accelerator |
| **On-Prem** | GroqRack available by request (millions of dollars) |
| **Availability** | GroqCard available for purchase. GroqRack for enterprises. |
| **Key Differentiator** | Deterministic, ultra-low latency inference. SRAM-only = no memory bandwidth bottleneck. |

**Assessment:** The LPU architecture is optimized for latency, not memory capacity. Only 230 MB SRAM per chip means a 70B model requires hundreds of chips working in coordination (576 LPUs for Llama 2 70B). A single $20K card cannot run useful models alone. GroqRack (full systems) costs millions. **Not viable for individual use.**

Sources: [Groq LPU Architecture](https://groq.com/lpu-architecture), [Groq $20K LPU Card](https://cryptoslate.com/groq-20000-lpu-card-breaks-ai-performance-records-to-rival-gpu-led-industry/), [Groq Pricing](https://groq.com/pricing)

---

## 6. SambaNova

### SambaNova SN40L RDU (Reconfigurable Dataflow Unit)

| Spec | Value |
|------|-------|
| **Architecture** | Dataflow with 3-tier memory (SRAM + HBM + DRAM) |
| **On-Chip SRAM** | 520 MB |
| **Compute (BF16)** | 640 TFLOPS |
| **Process** | TSMC 5nm |
| **Transistors** | 102 billion per socket |
| **Model Capacity** | Up to 5 trillion parameters per system node |
| **Power** | ~10 kW per rack (vs 140 kW for NVIDIA equivalent) |
| **Price** | Enterprise pricing only; sold as full-stack platform |
| **Availability** | Available as on-premises subscription or purchase |
| **Key Differentiator** | 3-tier memory enables serving massive models. 14x lower power than NVIDIA. Full-stack solution. |

**Assessment:** Enterprise-grade full-stack AI platform. Sold as a complete solution, not individual components. Pricing not disclosed but clearly enterprise-level (hundreds of thousands to millions). The 10 kW vs 140 kW power comparison is impressive. **Not accessible for individual use.**

Sources: [SambaNova SN40L](https://sambanova.ai/products/sn40l-rdu-ai-chip), [SN40L Best for Inference](https://sambanova.ai/blog/sn40l-chip-best-inference-solution), [SambaNova Platform](https://sambanova.ai)

---

## 7. Tenstorrent

### Tenstorrent Blackhole p150a (Consumer/Developer)

| Spec | Value |
|------|-------|
| **Memory** | 32 GB GDDR6 |
| **Memory Bandwidth** | ~768 GB/s (estimated based on GDDR6 config) |
| **Compute (FP8)** | 664 TFLOPS |
| **Cores** | 120 Tensix cores |
| **TDP** | Up to 300W |
| **Interconnect** | 4x QSFP-DD 800G ports (link multiple cards) |
| **Price** | $1,399 |
| **Availability** | Available for pre-order / shipping |
| **Key Differentiator** | Incredible price/performance. Open-source software stack. RISC-V based. Multi-card scalability via high-speed interconnect. Jim Keller's company. |

**Assessment:** The most accessible non-NVIDIA/non-Apple option at $1,399. 32 GB GDDR6 is limiting for large models but multiple cards can be linked via 800G interconnect (unlike consumer NVIDIA GPUs). However: the software ecosystem is extremely immature. Running common LLM inference engines (llama.cpp, vLLM, Ollama) is not straightforward. The Tensix architecture requires specific compilation. Early adopter territory. **Interesting for experimentation, risky for production use. Worth monitoring closely.**

Sources: [Tenstorrent Blackhole](https://tenstorrent.com/en/hardware/blackhole), [Tom's Hardware Tenstorrent](https://www.tomshardware.com/pc-components/gpus/tenstorrents-risc-v-based-wormhole-ai-accelerators-are-available-for-pre-order-today-pre-built-workstations-start-at-dollar12000), [Blackhole QuietBox Review](https://www.theregister.com/2025/11/27/tenstorrent_quietbox_review/)

---

### Tenstorrent Blackhole p100a (Consumer/Developer, Entry)

| Spec | Value |
|------|-------|
| **Memory** | 28 GB GDDR6 |
| **Compute** | 120 Tensix cores |
| **TDP** | Up to 300W |
| **Price** | $999 |
| **Availability** | Available |
| **Key Differentiator** | Sub-$1,000 AI accelerator card. |

Source: [Tenstorrent Hardware](https://tenstorrent.com/en/hardware/blackhole)

---

### Tenstorrent Wormhole n300 (Developer)

| Spec | Value |
|------|-------|
| **Memory** | 24 GB GDDR6 (aggregated) |
| **Memory Bandwidth** | 576 GB/s |
| **Compute (FP8)** | 466 TFLOPS |
| **TDP** | 300W |
| **Price** | $1,399 |
| **Availability** | Available |
| **Key Differentiator** | Previous-generation Tenstorrent card. Dual Wormhole processors. |

Source: [Wormhole](https://tenstorrent.com/en/hardware/wormhole), [AnandTech Wormhole Launch](https://www.anandtech.com/show/21482/tenstorrent-launches-wormhole-ai-processors-466-fp8-tflops-at-300w)

---

### Tenstorrent Workstations

| Product | Config | Price |
|---------|--------|-------|
| **QuietBox** | 4x Blackhole cards | ~$12,000+ |
| **TT-LoudBox** | 4x Wormhole n300 + 2x Xeon 4309Y + 512 GB RAM | ~$12,000+ |

**Assessment:** The QuietBox is the closest Tenstorrent product to a DGX Spark competitor. 4x Blackhole p150a = 128 GB GDDR6 total. At ~$12K, it's competitive on specs but the software maturity question remains. The Register's review was mixed — promising hardware, work-in-progress software.

Source: [Tenstorrent Developer Kits](https://tenstorrent.com/en/vision/tenstorrent-launches-next-generation-wormhole-based-developer-kits-and-workstations)

---

## 8. Google TPU

### Google TPU v6 (Trillium)

| Spec | Value |
|------|-------|
| **Memory** | Up to 144 GB HBM3 per chip |
| **Compute** | 4.7x peak compute over TPU v5e |
| **Energy Efficiency** | 67% more efficient than TPU v5e |
| **Scale** | Up to 256 TPUs per pod; 91 exaflops in a single cluster |
| **Price** | Cloud pricing via Google Cloud; on-prem pricing not public |
| **Availability** | GA on Google Cloud. On-prem sales began 2025. |
| **Key Differentiator** | Google began selling TPUs for on-premises deployment in 2025 — a major strategic shift. |

**Assessment:** Google selling TPUs on-prem is a significant development, but this is targeted at hyperscalers (Anthropic bought 400,000 units). Not available as individual cards or small-scale systems. Pricing is enterprise-level. **Not accessible for individual/SOHO use.**

Sources: [Google Trillium TPU](https://cloud.google.com/blog/products/compute/introducing-trillium-6th-gen-tpus), [Google TPU On-Prem](https://www.outlookbusiness.com/start-up/deeptech/google-challenges-nvidia-offers-tpus-for-on-premise-data-centers-meta-in-multi-billion-dollar-talks)

---

### Google Coral Edge TPU (Edge Device)

| Spec | Value |
|------|-------|
| **Compute** | 4 TOPS (INT8) |
| **Form Factor** | USB dongle or M.2 module |
| **Price** | ~$60-75 |
| **Availability** | Available now |
| **Key Differentiator** | Tiny, cheap edge AI accelerator for embedded/IoT. |

**Assessment:** 4 TOPS is useful for small vision models on embedded devices, not for LLM inference. **Completely insufficient for our use case.**

Source: [Google Coral](https://www.amazon.com/Google-Coral-Accelerator-coprocessor-Raspberry/dp/B07R53D12W)

---

## 9. Amazon Trainium / Inferentia

### AWS Trainium3 (Data Center, December 2025)

| Spec | Value |
|------|-------|
| **Memory** | 144 GB HBM3e per chip |
| **Memory Bandwidth** | 4,900 GB/s |
| **Compute (FP8)** | 2,520 TFLOPS (2.52 PFLOPs) |
| **UltraServer** | 144 chips / 362 FP8 PFLOPs / 20.7 TB HBM3e |
| **Price** | AWS cloud only; on-prem not available |
| **Availability** | Available as EC2 instances |
| **Key Differentiator** | 40% better energy efficiency over Trainium2. Supports MXFP4 and structured sparsity. Future Trainium4 will support NVIDIA NVLink Fusion for hybrid deployments. |

**Assessment:** Cloud-only. Amazon does not sell Trainium/Inferentia chips for on-premises use. **Not viable for local deployment.**

Sources: [AWS Trainium3](https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-ec2-trn3-ultraservers/), [Next Platform Trainium4](https://www.nextplatform.com/2025/12/03/with-trainium4-aws-will-crank-up-everything-but-the-clocks/)

---

### AWS Inferentia2 (Data Center Inference)

| Spec | Value |
|------|-------|
| **Memory** | 32 GB HBM per chip |
| **Compute (FP16)** | 190 TFLOPS |
| **Max Config** | 12 chips per Inf2 instance (384 GB, 9.8 TB/s aggregate) |
| **Price** | Cloud pricing only (EC2 Inf2 instances) |
| **Availability** | GA on AWS |
| **Key Differentiator** | 4x throughput, 10x lower latency over Inferentia1. |

**Assessment:** Cloud-only, like all AWS custom silicon. **Not viable for local deployment.**

Sources: [AWS Inferentia](https://aws.amazon.com/ai/machine-learning/inferentia/), [EC2 Inf2](https://aws.amazon.com/ec2/instance-types/inf2/)

---

## 10. Chinese Alternatives

### Huawei Ascend 910C (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 128 GB HBM3 |
| **Memory Bandwidth** | 3,200 GB/s |
| **Compute (FP16)** | 800 TFLOPS |
| **Compute (INT8)** | ~1,600 TOPS |
| **Process** | SMIC 7nm (DUV) |
| **TDP** | Not disclosed |
| **Price** | ~180,000-200,000 RMB (~$25,000-28,000 USD) |
| **Availability** | Large-scale shipments in China. Export-restricted. |
| **Key Differentiator** | China's most powerful AI chip. Dual-910B die design. ~60-70% of H100 performance. |

**Assessment:** Export-restricted — cannot be purchased outside China. Even within China, supply is constrained (30% yield rate). Software ecosystem (CANN/MindSpore) is incompatible with CUDA. **Not accessible outside China.**

Sources: [Huawei Ascend 910C vs H100](https://www.nexgen-compute.com/blog/huawei-ascend-910c-vs-nvidia-h100-ai-chip-comparison), [Huawei Ascend Specs](https://www.bitrue.com/blog/huawei-ascend-ai-chip-specs-2025), [TrendForce Huawei DeepSeek](https://www.trendforce.com/news/2025/04/29/news-decoding-huaweis-deepseek-all-in-one-machine-60-70-of-nvidia-h100-performance-at-an-appealing-price/)

---

### Biren BR100 (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 64 GB HBM2E |
| **Memory Bandwidth** | 896 GB/s |
| **Compute (FP32)** | 256 TFLOPS |
| **Compute (INT8)** | 2,048 TOPS |
| **Process** | TSMC 7nm |
| **Transistors** | 77 billion |
| **TDP** | 550W |
| **Price** | Not publicly disclosed |
| **Availability** | Limited, primarily in China. Biren IPO on HKEX in Jan 2026. |
| **Key Differentiator** | 2.6x average speedup over NVIDIA A100. Dual chiplet design. |

**Assessment:** Chinese company, primarily serving Chinese market. Export restrictions on advanced chips affect availability outside China. TSMC-manufactured (unlike Huawei which uses SMIC), so potentially less restricted, but practical availability in Europe is negligible. **Not accessible for Swiss purchase.**

Sources: [Biren BR100 Details](https://www.techpowerup.com/298098/biren-br100-detailed-chinas-ai-hpc-processor-storms-into-the-hpc-gpu-big-leagues), [Chips and Cheese BR100](https://chipsandcheese.com/p/hot-chips-34-birens-br100-a-machine-learning-gpu-from-china)

---

### Moore Threads MTT S4000 (Data Center)

| Spec | Value |
|------|-------|
| **Memory** | 48 GB VRAM |
| **Memory Bandwidth** | 768 GB/s |
| **Compute (FP32)** | 25 TFLOPS |
| **Compute (INT8)** | 200 TOPS |
| **Interface** | PCIe Gen5 x16 |
| **Cores** | 128 Tensor Cores |
| **Price** | Not disclosed |
| **Availability** | China-focused. Large-scale cluster deployments. |
| **Key Differentiator** | MUSA architecture with zero-cost CUDA framework translation. Supports DeepSeek models. |

**Assessment:** China-focused company. Products not available for international purchase. CUDA-translation layer is promising for software compatibility but untested outside China. **Not accessible.**

Sources: [Moore Threads S4000](https://en.mthreads.com/product/S4000), [VideoCardz S4000](https://videocardz.com/newz/moore-threads-introduces-mtt-s4000-48gb-ai-gpu-with-mtlink-and-zero-cost-nvidia-cuda-framework-translation)

---

### Baidu Kunlun (Upcoming)

| Spec | Value |
|------|-------|
| **Current** | Kunlun P800 (30,000-card cluster deployed) |
| **Upcoming** | M100 (inference-optimized, early 2026) / M300 (training+inference, early 2027) |
| **Price** | Not disclosed |
| **Availability** | China-only; Kunlunxin planning HKEX IPO |
| **Key Differentiator** | M100 offers 3.5x single-card token throughput increase for mainstream model inference. |

**Assessment:** China-only. Kunlunxin (the chip subsidiary) is filing for Hong Kong listing. **Not accessible internationally.**

Sources: [Baidu Kunlun Chips](http://www.ecns.cn/news/sci-tech/2025-11-14/detail-ihewyezc9800878.shtml), [TrendForce Baidu Roadmap](https://www.trendforce.com/news/2025/11/13/news-baidu-rolls-out-kunlun-roadmap-m100-m300-ai-chips-arrive-2026-2027/)

---

## 11. AI Chip Startups

### Etched Sohu (Transformer ASIC)

| Spec | Value |
|------|-------|
| **Architecture** | Transformer-only ASIC |
| **Memory** | HBM3E (capacity not disclosed) |
| **Process** | TSMC 4nm |
| **Performance Claim** | 500,000 tok/s on Llama-70B (8-chip server) vs 23,000 tok/s for 8x H100 |
| **Price** | Not disclosed |
| **Availability** | **Not yet shipping.** $500M funding secured for TSMC 4nm fab capacity. |
| **Key Differentiator** | 10-20x faster and cheaper than H100 for transformer inference (claimed). |

**Assessment:** Extraordinary claims but product has not shipped to customers. Hardware-coded for transformer architecture only — future model architectures (state-space, etc.) would not be supported. Very high risk. **Vaporware until proven otherwise.**

Sources: [Tom's Hardware Sohu](https://www.tomshardware.com/tech-industry/artificial-intelligence/sohu-ai-chip-claimed-to-run-models-20x-faster-and-cheaper-than-nvidia-h100-gpus), [Etched $500M Raise](https://ai2.work/technology/etcheds-500-million-raise-a-blueprint-for-enterprise-ai-inference-in-2026/)

---

### Positron AI Atlas / Asimov (Inference ASIC)

| Spec | Value |
|------|-------|
| **Atlas (Current)** | 8x Archer accelerators, 256 GB total (32 GB HBM each) |
| **Performance** | 280 tok/s/user on Llama 3.1 8B at 2,000W vs 180 tok/s for 8x H200 at 5,900W |
| **Price** | Not disclosed (enterprise sales) |
| **Asimov (2026-2027)** | 2 TB LPDDR5X per chip; target 16T parameter models on single machine |
| **Process** | TSMC N4/N5 |
| **Funding** | $230M Series B (Feb 2026), $1B valuation |
| **Availability** | Atlas available to enterprises. Asimov targeting early 2027. |
| **Key Differentiator** | 3.5x better performance-per-dollar and 66% lower power than H100. Made in USA (TSMC Arizona). |

**Assessment:** Enterprise-level product. Atlas systems are not sold to individuals. The Asimov chip with 2 TB LPDDR5X is a fascinating future direction. **Not accessible for individual purchase. Future watch for Asimov.**

Sources: [Positron Atlas](https://www.positron.ai/atlas), [Positron $230M Series B](https://techcrunch.com/2026/02/04/exclusive-positron-raises-230m-series-b-to-take-on-nvidias-ai-chips/), [Tom's Hardware Positron](https://www.tomshardware.com/tech-industry/artificial-intelligence/positron-ai-says-its-atlas-accelerator-beats-nvidia-h200-on-inference-in-just-33-percent-of-the-power-delivers-280-tokens-per-second-per-user-with-llama-3-1-8b-in-2000w-envelope)

---

### d-Matrix Corsair (In-Memory Compute)

| Spec | Value |
|------|-------|
| **Memory** | 2 GB SRAM + 256 GB LPDDR5X per card |
| **Memory Bandwidth** | 150 TB/s (on-chip) |
| **Compute (INT8)** | 2,400 TOPS |
| **Process** | TSMC 6nm |
| **Form Factor** | PCIe 5.0 x16 |
| **Performance** | 2ms per output token on Llama3-70B |
| **Price** | Not disclosed |
| **Availability** | Sampling to early-access customers; broad availability Q2 2025 |
| **Key Differentiator** | Digital in-memory computing. 38 TOPS/watt efficiency. 256 GB per card. |

**Assessment:** Very promising specs — 256 GB per card and 2ms/token on 70B would be revolutionary. However: early-access only, unproven in real-world deployment, no retail availability. The in-memory compute approach is novel but untested at scale. **High-potential future option, not actionable now.**

Sources: [d-Matrix Corsair](https://www.d-matrix.ai/product/), [ServeTheHome d-Matrix](https://www.servethehome.com/d-matrix-corsair-in-memory-computing-for-ai-inference-at-hot-chips-2025/), [d-Matrix $275M Raise](https://siliconangle.com/2025/11/12/chip-startup-d-matrix-raises-275m-speed-inference-memory-compute/)

---

### FuriosaAI RNGD (Data Center Inference)

| Spec | Value |
|------|-------|
| **Memory** | 2x HBM3 modules |
| **Memory Bandwidth** | 1,500 GB/s |
| **TDP** | 180W |
| **Interface** | PCIe Gen5 x16 |
| **Multi-tenancy** | Single chip acts as 2, 4, or 8 isolated NPUs |
| **Price** | Not disclosed |
| **Availability** | Mass production since late 2025. NXT RNGD Server available. |
| **Key Differentiator** | 2.25x inference performance-per-watt vs GPUs. 180W is remarkably low for this performance class. |

**Assessment:** Data center product. Sold as part of NXT RNGD Server systems. Not available as individual PCIe cards for consumer purchase. Software ecosystem is proprietary (FuriosaAI SDK). **Not accessible for individual use.**

Sources: [FuriosaAI RNGD](https://furiosa.ai/rngd), [FuriosaAI $125M Funding](https://evertiq.com/news/2025-08-11-furiosaai-closes-125m-round-to-scale-production-of-ai-inference-chip)

---

### MatX (Stealth, ex-Google)

| Spec | Value |
|------|-------|
| **Architecture** | Single large processing core, cost-efficiency focused |
| **Performance Claim** | <100th of a second per token on 70B models |
| **Scale** | Clusters of hundreds of thousands of chips |
| **Price** | Not disclosed |
| **Availability** | First product expected finalized by 2025 (no public shipping confirmation) |
| **Funding** | $130M total |
| **Key Differentiator** | Founded by ex-Google TPU engineers. Single-purpose chip design. |

**Assessment:** Stealth-mode startup. No shipping product confirmed. **Not actionable.**

Sources: [MatX](https://matx.com/), [TechCrunch MatX](https://techcrunch.com/2024/11/22/ai-chip-startup-matx-founded-by-google-alums-raises-series-a-at-300m-valuation-sources-say/)

---

### Mythic AI (Analog Compute)

| Spec | Value |
|------|-------|
| **Architecture** | Analog Matrix Processor (AMP) with flash memory |
| **Performance** | Up to 25 TOPS |
| **Efficiency** | 100x more energy-efficient than GPUs (claimed) |
| **Price** | Not disclosed |
| **Availability** | Pre-production; $125M funding secured Dec 2025 |
| **Key Differentiator** | Analog compute in flash memory. 10x cost savings, 3.8x lower power. |

**Assessment:** Edge AI focus (robotics, automotive, defense). 25 TOPS is far too low for LLM inference. Interesting technology but **not applicable to our workload.**

Sources: [Mythic AI](https://mythic.ai/), [Mythic $125M Raise](https://mythic.ai/whats-new/mythic-to-challenge-ais-gpu-pantheon-with-100x-energy-advantage-and-oversubscribed-125m-raise/)

---

### Graphcore/SoftBank IPU (Intelligence Processing Unit)

| Spec | Value |
|------|-------|
| **Current IPU** | Colossus MK2 GC200: 1,472 cores, 900 MB in-processor memory, 250 TFLOPS FP16 |
| **System** | IPU-M2000: 1 petaFLOP, 3.6 GB in-processor memory |
| **Interconnect** | IPU-Fabric: 2.8 Tbps |
| **Upcoming** | "Izanagi" chips (2026) combining Ampere CPUs + Graphcore IPUs |
| **Price** | Not publicly priced |
| **Availability** | Acquired by SoftBank (2024). First Izanagi deployments expected 2026. |
| **Key Differentiator** | Part of SoftBank's "Silicon Trinity" (Arm + Ampere + Graphcore). 30% TCO reduction for LLM inference (claimed). |

**Assessment:** SoftBank acquisition is repositioning Graphcore for hyperscale deployment (Stargate data centers). Not available as individual products. The IPU architecture's in-processor memory model is interesting but current capacity (3.6 GB) is too small for large LLMs. Izanagi chips in 2026 may change the landscape. **Not accessible for individual use. Watch Izanagi.**

Sources: [Graphcore IPU Products](https://www.graphcore.ai/products/ipu), [SoftBank Graphcore Integration](https://markets.financialcontent.com/wedbush/article/tokenring-2025-12-31-softbanks-ai-vertical-play-integrating-ampere-and-graphcore-to-challenge-the-gpu-giants)

---

### Esperanto Technologies (Defunct)

Esperanto Technologies shut down in July 2025. Their RISC-V-based ET-SoC-1 (1,088 RISC-V cores) IP was acquired by Ainekko for open-source development. **Dead.**

Source: [Esperanto Exits](https://xpu.pub/2025/07/05/esperanto/)

---

## 12. FPGA-Based Solutions

### AMD Alveo V80 (FPGA Compute Accelerator)

| Spec | Value |
|------|-------|
| **Memory** | 32 GB HBM2e |
| **FPGA Fabric** | AMD Versal HBM XCV80 |
| **Connectivity** | 4x QSFP56 |
| **Price** | $9,495 MSRP |
| **Availability** | Available now |
| **Key Differentiator** | Programmable accelerator with HBM. Adaptable to changing workloads. |

**Assessment:** FPGAs are reprogrammable but typically deliver lower peak performance than purpose-built ASICs or GPUs for AI inference. The V80 at $9,495 with 32 GB HBM2e offers less inference performance and less memory than an RTX PRO 6000 at a similar price. FPGAs excel in low-latency, custom-pipeline scenarios (financial trading, video processing) but are not competitive for general LLM inference. **Not suitable for our workload.**

Sources: [AMD Alveo V80](https://www.amd.com/en/products/accelerators/alveo/v80.html), [ServeTheHome Alveo V80](https://www.servethehome.com/new-faster-amd-alveo-v80-accelerator-with-hbm2e-and-fast-networking/)

---

### Intel Stratix 10 NX (AI-Optimized FPGA)

| Spec | Value |
|------|-------|
| **Architecture** | AI Tensor Blocks with integrated HBM |
| **Process** | Intel 14nm |
| **Performance** | 15x more INT8 compute vs Stratix 10 MX |
| **Price** | Enterprise pricing |
| **Availability** | Available |
| **Key Differentiator** | First AI-optimized FPGA from Intel. |

**Assessment:** Older technology (14nm). Similar limitations as AMD Alveo — FPGAs are not competitive for general LLM inference. **Not suitable.**

Source: [Intel Stratix 10 NX](https://www.hpcwire.com/2020/06/18/intel-debuts-stratix-10-nx-fpgas-targeting-ai-workloads/)

---

## 13. Multi-GPU Consumer Setups

### 4x NVIDIA RTX 5090 Configuration

| Spec | Value |
|------|-------|
| **Total VRAM** | 128 GB GDDR7 (4x 32 GB) |
| **Total Bandwidth** | 7,168 GB/s (aggregate, but cross-GPU via PCIe) |
| **Performance** | 4,622 tok/s on quantized small models; ~27 tok/s on 70B Q4 (2x) |
| **TDP** | 2,300W (GPUs alone) |
| **Price** | ~$8,000-15,200 (GPUs) + $3,000-5,000 (system) = ~$11,000-20,000 total |
| **Platform** | Requires WRX90 (Threadripper PRO) for 4x x16 lanes |
| **Availability** | Limited stock, prices 25-90% above MSRP |
| **Key Differentiator** | Highest absolute consumer throughput. |

**Assessment for our use case:** 4x RTX 5090 gives 128 GB VRAM — matching our 96 GB minimum with headroom. But critical problems:

1. **2,300W GPU power** = requires 3,000W PSU, special wiring, extreme cooling. Not home-office friendly.
2. **PCIe bottleneck**: Without NVLink, cross-GPU communication adds 20-40% overhead for tensor parallelism.
3. **Cost**: CHF 11,000-20,000 all-in for 128 GB that's split across 4 cards, vs CHF 9,500-12,000 for a single RTX PRO 6000 build with 96 GB on one card.
4. **Noise**: 4x 575W cards = deafening in a home office (55+ dBA).

**Verdict: For running independent models on separate GPUs (multi-agent), a 2x RTX 5090 setup (~64 GB, ~CHF 6,000-9,000) is more practical. For single-model inference, the RTX PRO 6000 wins decisively due to unified 96 GB memory.**

Sources: [RTX 5090 LLM Benchmarks](https://www.runpod.io/blog/rtx-5090-llm-benchmarks), [BIZON G3000](https://bizon-tech.com/bizon-g3000.html), [LocalLLM GPU Guide](https://localllm.in/blog/best-gpus-llm-inference-2025)

---

### 2x NVIDIA RTX 5090 Configuration

| Spec | Value |
|------|-------|
| **Total VRAM** | 64 GB GDDR7 |
| **Interconnect** | PCIe 5.0 only (no NVLink) |
| **Performance** | ~27 tok/s on 70B Q4 (tensor parallel), ~150+ tok/s each on 8B models |
| **TDP** | 1,150W (GPUs) |
| **Price** | ~CHF 4,800-7,000 (GPUs) + ~CHF 2,000-3,000 (system) |
| **Platform** | TRX50 or AM5 X870E |

**Assessment:** 64 GB is below our 96 GB minimum for multi-agent MoE workloads. Good for running two independent smaller models simultaneously. **Viable as a supplement to RTX PRO 6000, not as a standalone solution.**

Source: [Dual RTX 5090 vs H100](https://www.hardware-corner.net/dual-rtx-5090-vs-h100-for-llm/)

---

## 14. Purpose-Built Inference Appliances

### NVIDIA DGX Spark (Reference Point)

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

**Assessment for our use case:** The DGX Spark is the benchmark for desktop AI appliances. 128 GB unified memory is attractive, but 273 GB/s bandwidth is severely limiting — roughly 7x slower than RTX PRO 6000. Estimated 70B Q4 inference: ~7-8 tok/s. Good for experimentation and smaller models, not for production 60-80 tok/s coding workflows. **Too slow for our performance targets.**

Sources: [NVIDIA DGX Spark](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/), [DGX Spark Prices](https://www.glukhov.org/post/2025/10/nvidia-dgx-spark-prices/)

---

### NVIDIA DGX Station (Upcoming, Spring 2026)

| Spec | Value |
|------|-------|
| **Chip** | GB300 Grace Blackwell Ultra Desktop Super |
| **Memory** | 784 GB unified (288 GB GPU HBM3e + system RAM) |
| **Compute** | 20 PFLOPS AI |
| **Price** | Not disclosed (likely $50,000+) |
| **Availability** | Spring 2026 via ASUS, Dell, HP, MSI, Supermicro, others |
| **Key Differentiator** | Desktop powerhouse. 784 GB unified memory. 20x DGX Spark performance. |

**Assessment:** This is the ultimate desktop AI machine. 784 GB memory with 288 GB HBM3e bandwidth would demolish our performance targets. But pricing will almost certainly be $50,000-100,000+ based on NVIDIA's positioning. **Dream machine, likely far above budget.**

Source: [DGX Station Blog](https://blogs.nvidia.com/blog/dgx-spark-and-station-open-source-frontier-models/)

---

### ASUS Ascent GX10 (Strix Halo Appliance)

| Spec | Value |
|------|-------|
| **Chip** | AMD Ryzen AI MAX+ 395 (Strix Halo) |
| **Memory** | Up to 128 GB LPDDR5X (1 TB storage) |
| **Memory Bandwidth** | 256 GB/s |
| **Price** | $3,000 (pre-order, 1 TB model) |
| **Availability** | Pre-order available |
| **Key Differentiator** | DGX Spark competitor at lower price. x86 architecture. |

**Assessment:** Same bandwidth limitations as DGX Spark. ROCm software ecosystem. $3,000 for 128 GB is good value, but inference speed will be ~7-8 tok/s on 70B. **Same verdict as DGX Spark: too slow for production coding inference.**

Source: [DGX Spark Alternatives](https://research.aimultiple.com/dgx-spark-alternatives/)

---

### Lenovo Inference Servers (Enterprise)

Lenovo announced purpose-built AI inferencing servers at CES 2026. Enterprise-grade rack servers, not consumer products. **Not accessible for home use.**

Source: [Lenovo AI Inferencing](https://news.lenovo.com/pressroom/press-releases/lenovo-revolutionizes-real-time-enterprise-ai-with-new-inferencing-servers/)

---

### Tenstorrent QuietBox (Developer Workstation)

| Spec | Value |
|------|-------|
| **Accelerators** | 4x Blackhole cards (~128 GB GDDR6 total) |
| **CPU** | Intel Xeon |
| **Price** | ~$12,000+ |
| **Availability** | Available |
| **Key Differentiator** | Non-NVIDIA AI workstation at reasonable price. 800G card-to-card interconnect. |

**Assessment:** Covered in Section 7. Software ecosystem immaturity is the primary risk. **Interesting for experimentation, not for production.**

Source: [Blackhole QuietBox Review](https://www.theregister.com/2025/11/27/tenstorrent_quietbox_review/)

---

## 15. Photonic / Emerging Technologies

### Lightmatter Passage M1000 (Photonic Interconnect)

| Spec | Value |
|------|-------|
| **Technology** | 3D photonic interposer for GPU interconnect |
| **Bandwidth** | 114 Tbps total optical bandwidth |
| **Size** | >4,000 mm^2 multi-reticle active photonic interposer |
| **Availability** | Summer 2025 (for system integrators) |
| **Key Differentiator** | Optical interconnect enabling thousands of GPUs in a single domain. Not a standalone compute product. |

**Assessment:** Lightmatter makes interconnects, not compute accelerators. Enables better GPU-to-GPU communication but requires massive scale to be relevant. **Not a standalone inference solution.**

Sources: [Lightmatter M1000](https://lightmatter.co/press-release/lightmatter-unveils-passage-m1000-photonic-superchip-worlds-fastest-ai-interconnect/), [Lightmatter L200](https://lightmatter.co/press-release/lightmatter-announces-passage-l200-the-fastest-co-packaged-optics-for-ai/)

---

### Hailo AI Accelerators (Edge)

| Spec | Value |
|------|-------|
| **Hailo-8** | 26 TOPS INT8, M.2 module |
| **Hailo-10H** | 40 TOPS INT4, M.2 module, GenAI support |
| **Price** | ~$100-300 per module |
| **Availability** | Available now |
| **Key Differentiator** | Plug-and-play edge AI. M.2 form factor. |

**Assessment:** Edge AI accelerators for computer vision and small models. 26-40 TOPS is negligible for LLM inference. **Not applicable to our use case.**

Sources: [Hailo AI Accelerators](https://hailo.ai/products/ai-accelerators/), [Hailo-10H](https://hailo.ai/products/ai-accelerators/hailo-10h-ai-accelerator/)

---

## 16. Summary Comparison Table

### Platforms Accessible for Individual/SOHO Purchase (Feb 2026)

| Platform | Memory | Bandwidth | Est. 70B Q4 tok/s | Price (USD) | Software Maturity | Verdict |
|----------|--------|-----------|-------------------|-------------|-------------------|---------|
| **NVIDIA RTX PRO 6000** | 96 GB GDDR7 | 1,792 GB/s | ~32 | ~$7,500 | CUDA (excellent) | **Best option** |
| **NVIDIA RTX 5090 (2x)** | 64 GB GDDR7 | 2x 1,792 GB/s | ~27 (TP) | ~$4,000-7,000 | CUDA (excellent) | Good for multi-agent |
| **NVIDIA DGX Spark** | 128 GB unified | 273 GB/s | ~7-8 | $3,999 | CUDA (excellent) | Too slow |
| **AMD Radeon PRO W7900** | 48 GB GDDR6 | 864 GB/s | ~24 | ~$4,000 | ROCm (improving) | Bandwidth-limited |
| **AMD Strix Halo 128GB** | 128 GB unified | 256 GB/s | ~7-8 | ~$1,500-2,000 | ROCm (immature) | Too slow, great value |
| **Apple Mac Studio M4 Ultra** | 192 GB unified | 819 GB/s | ~22 | ~$5,000-8,000 | MLX (good) | Good bandwidth, no CUDA |
| **Tenstorrent Blackhole p150a** | 32 GB GDDR6 | ~768 GB/s | Unknown | $1,399 | Immature | Experimental only |
| **Intel Arc Pro B60** | 24 GB GDDR6 | 456 GB/s | Unknown | ~$500-1,000 | Immature | Q4 2025 launch |
| **Intel Arc Pro B50** | 16 GB GDDR6 | 224 GB/s | N/A | $349 | Immature | Too small |

### Platforms NOT Accessible for Individual Purchase

| Platform | Why Not | Memory | Performance Class |
|----------|---------|--------|-------------------|
| AMD Instinct MI300X/325X/350X | Data center only, passive cooling, not sold retail | 192-288 GB HBM | H100/H200 class |
| Intel Gaudi 3 | Enterprise OEM only, $125K for 8-chip board | 128 GB HBM2E | H100 competitive |
| Intel Crescent Island | Not shipping until H2 2026+ | 160 GB LPDDR5X | TBD |
| Qualcomm Cloud AI 100 Ultra | Enterprise, proprietary software | 128 GB | Mid-range |
| Cerebras CS-3 | Multi-million dollar systems | 44 GB SRAM (on-chip) | Beyond H100 |
| Groq LPU | $20K/card, needs hundreds for one model | 230 MB SRAM/chip | Ultra-low latency |
| SambaNova SN40L | Enterprise full-stack only | 520 MB SRAM + HBM + DRAM | Enterprise class |
| Google TPU v6 | Enterprise bulk sales only | 144 GB HBM3 | H100+ class |
| AWS Trainium3 | Cloud-only | 144 GB HBM3e | H100+ class |
| AWS Inferentia2 | Cloud-only | 32 GB HBM | Mid-range |
| Huawei Ascend 910C | China-only, export restricted | 128 GB HBM3 | ~70% H100 |
| Biren BR100 | China-focused | 64 GB HBM2E | A100+ class |
| Moore Threads S4000 | China-focused | 48 GB | A100 class |
| Baidu Kunlun | China-only | Not disclosed | Mid-range+ |
| Etched Sohu | Not yet shipping | HBM3E | Claims 20x H100 |
| Positron Atlas/Asimov | Enterprise only | 256 GB / 2 TB (Asimov) | H200+ class |
| d-Matrix Corsair | Early access only | 256 GB LPDDR5X | Promising |
| FuriosaAI RNGD | Enterprise server only | HBM3 | Mid-range+ |
| MatX | Not shipping | TBD | Claims sub-ms 70B |
| Graphcore IPU (SoftBank) | Acquisition transition, no retail | 900 MB in-processor | Specialized |

---

## 17. Relevance Assessment for Our Use Case

### Requirements Recap
- Budget: CHF 4,000-15,000
- Performance: 60-80 tok/s (MoE), 30 tok/s minimum (dense 70B)
- Memory: 96 GB minimum GPU-accessible
- Environment: Home office, quiet, standard power
- Software: CUDA preferred, active inference ecosystem

### Conclusion

After surveying **30+ hardware platforms** across 15 categories, the competitive landscape for **local AI inference accessible to individuals in February 2026** is remarkably narrow:

1. **NVIDIA dominates the accessible market.** The RTX PRO 6000 Blackwell (96 GB, 1,792 GB/s, CUDA) remains the best option for our use case by a significant margin. No competitor matches its combination of memory capacity, bandwidth, software maturity, and reasonable power envelope (300W).

2. **Apple Silicon is the only mature non-NVIDIA alternative.** The Mac Studio M4 Ultra (192 GB, 819 GB/s) offers more memory at lower bandwidth with excellent MLX software support, but no CUDA.

3. **AMD workstation GPUs are bandwidth-limited.** The Radeon PRO W7900 (48 GB, 864 GB/s) has half the bandwidth of the RTX PRO 6000 and an immature software stack.

4. **AMD Strix Halo and NVIDIA DGX Spark compete on memory capacity, not speed.** Both offer 128 GB unified memory at ~256-273 GB/s — enough to load large models but too slow for production inference (~7-8 tok/s on 70B).

5. **Tenstorrent is the most interesting wildcard** at $1,399/card with 800G interconnect, but the software ecosystem needs 1-2 more years to mature.

6. **Intel Crescent Island (160 GB, air-cooled, inference-optimized) is the most promising future competitor** — but it won't ship until late 2026 at the earliest.

7. **Every other platform** is either enterprise-only, cloud-only, China-only, not yet shipping, or not powerful enough. The AI accelerator market is heavily bifurcated between massive data center products (Instinct, Gaudi, TPU, Trainium, Cerebras, Groq, SambaNova) and consumer/edge products that lack the memory and bandwidth for serious LLM inference.

8. **The CUDA software ecosystem moat is enormous.** Even platforms with competitive hardware (AMD, Intel, Tenstorrent) are held back by software maturity. CUDA's compatibility with vLLM, llama.cpp, Ollama, ExLlamaV2, and every other major inference engine is a decisive advantage.

**Bottom line: For local AI inference in February 2026, no alternative platform changes the recommendation made in the existing research. The RTX PRO 6000 Blackwell is the optimal choice within our constraints.**

---

## Sources

### AMD
- [AMD Instinct MI300 Series](https://www.amd.com/en/products/accelerators/instinct/mi300.html)
- [AMD Instinct Roadmap](https://ir.amd.com/news-events/press-releases/detail/1201/amd-accelerates-pace-of-data-center-ai-innovation-and-leadership-with-expanded-amd-instinct-gpu-roadmap)
- [AMD MI350 Series](https://www.amd.com/en/products/accelerators/instinct/mi350.html)
- [AMD Radeon PRO W7900](https://www.amd.com/en/products/graphics/workstations/radeon-pro/w7900.html)
- [AMD Strix Halo vs DGX Spark (The Register)](https://www.theregister.com/2025/12/25/amd_strix_halo_nvidia_spark/)
- [Beelink GTR9 Pro Review](https://www.servethehome.com/beelink-gtr9-pro-review-amd-ryzen-ai-max-395-system-with-128gb-and-dual-10gbe/)
- [Framework Desktop Review](https://www.servethehome.com/framework-desktop-review-a-solid-amd-strix-halo/)
- [AMD GPU Guide (BentoML)](https://www.bentoml.com/blog/amd-data-center-gpus-mi250x-mi300x-mi350x-and-beyond)
- [MI300X Pricing](https://getdeploying.com/gpus/amd-mi300x)

### Intel
- [Intel Gaudi 3](https://www.intel.com/content/www/us/en/products/details/processors/ai-accelerators/gaudi.html)
- [Intel Arc Pro B-Series (igor'sLAB)](https://www.igorslab.de/en/computex-2025-intel-arc-pro-b-series-new-gpus-for-workstation-and-ki-inference-at-a-glance/)
- [Intel Arc Pro B50 Review (Puget)](https://www.pugetsystems.com/labs/articles/intel-arc-pro-b50-review/)
- [Intel Arc Pro B60 vLLM](https://blog.vllm.ai/2025/11/11/intel-arc-pro-b.html)
- [Intel Crescent Island (Tom's Hardware)](https://www.tomshardware.com/pc-components/gpus/intel-unveils-crescent-island-an-inference-only-gpu-with-xe3p-architecture-and-160gb-of-memory)
- [Intel Crescent Island (Phoronix)](https://www.phoronix.com/review/intel-crescent-island)
- [Intel Gaudi 3 (Tom's Hardware)](https://www.tomshardware.com/tech-industry/artificial-intelligence/intel-launches-gaudi-3-accelerator-for-ai-slower-than-h100-but-also-cheaper)

### Qualcomm
- [Qualcomm Cloud AI 100 Ultra](https://www.qualcomm.com/artificial-intelligence/data-center/cloud-ai-100-ultra)
- [Qualcomm On-Prem AI Appliance](https://www.qualcomm.com/news/releases/2025/01/qualcomm-launches-on-prem-ai-appliance-solution-and-inference-su)
- [Snapdragon X2 Elite](https://www.qualcomm.com/news/releases/2025/09/new-snapdragon-x2-elite-extreme-and-snapdragon-x2-elite-are-the-)

### Cerebras
- [Cerebras Chip](https://www.cerebras.ai/chip)
- [Cerebras Inference](https://www.cerebras.ai/press-release/cerebras-launches-the-worlds-fastest-ai-inference)
- [IEEE Spectrum WSE-3](https://spectrum.ieee.org/cerebras-chip-cs3)

### Groq
- [Groq LPU Architecture](https://groq.com/lpu-architecture)
- [Groq Pricing](https://groq.com/pricing)
- [Groq $20K LPU Card](https://cryptoslate.com/groq-20000-lpu-card-breaks-ai-performance-records-to-rival-gpu-led-industry/)

### SambaNova
- [SambaNova SN40L](https://sambanova.ai/products/sn40l-rdu-ai-chip)
- [SN40L Best for Inference](https://sambanova.ai/blog/sn40l-chip-best-inference-solution)

### Tenstorrent
- [Tenstorrent Blackhole](https://tenstorrent.com/en/hardware/blackhole)
- [Tenstorrent Wormhole](https://tenstorrent.com/en/hardware/wormhole)
- [Tom's Hardware Tenstorrent Pre-order](https://www.tomshardware.com/pc-components/gpus/tenstorrents-risc-v-based-wormhole-ai-accelerators-are-available-for-pre-order-today-pre-built-workstations-start-at-dollar12000)
- [Blackhole QuietBox Review (The Register)](https://www.theregister.com/2025/11/27/tenstorrent_quietbox_review/)
- [AnandTech Wormhole Launch](https://www.anandtech.com/show/21482/tenstorrent-launches-wormhole-ai-processors-466-fp8-tflops-at-300w)

### Google TPU
- [Google Trillium TPU](https://cloud.google.com/blog/products/compute/introducing-trillium-6th-gen-tpus)
- [Google TPU On-Prem Sales](https://www.outlookbusiness.com/start-up/deeptech/google-challenges-nvidia-offers-tpus-for-on-premise-data-centers-meta-in-multi-billion-dollar-talks)

### Amazon
- [AWS Trainium3](https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-ec2-trn3-ultraservers/)
- [AWS Inferentia](https://aws.amazon.com/ai/machine-learning/inferentia/)
- [Next Platform Trainium4](https://www.nextplatform.com/2025/12/03/with-trainium4-aws-will-crank-up-everything-but-the-clocks/)

### Chinese Alternatives
- [Huawei Ascend 910C vs H100](https://www.nexgen-compute.com/blog/huawei-ascend-910c-vs-nvidia-h100-ai-chip-comparison)
- [Huawei Ascend Specs 2025](https://www.bitrue.com/blog/huawei-ascend-ai-chip-specs-2025)
- [Biren BR100 (TechPowerUp)](https://www.techpowerup.com/298098/biren-br100-detailed-chinas-ai-hpc-processor-storms-into-the-hpc-gpu-big-leagues)
- [Moore Threads S4000](https://en.mthreads.com/product/S4000)
- [Baidu Kunlun Chips](http://www.ecns.cn/news/sci-tech/2025-11-14/detail-ihewyezc9800878.shtml)

### Startups
- [Etched Sohu (Tom's Hardware)](https://www.tomshardware.com/tech-industry/artificial-intelligence/sohu-ai-chip-claimed-to-run-models-20x-faster-and-cheaper-than-nvidia-h100-gpus)
- [Positron Atlas](https://www.positron.ai/atlas)
- [Positron $230M Series B](https://techcrunch.com/2026/02/04/exclusive-positron-raises-230m-series-b-to-take-on-nvidias-ai-chips/)
- [d-Matrix Corsair](https://www.d-matrix.ai/product/)
- [d-Matrix $275M Raise](https://siliconangle.com/2025/11/12/chip-startup-d-matrix-raises-275m-speed-inference-memory-compute/)
- [FuriosaAI RNGD](https://furiosa.ai/rngd)
- [MatX](https://matx.com/)
- [Mythic AI](https://mythic.ai/)
- [Graphcore/SoftBank Integration](https://markets.financialcontent.com/wedbush/article/tokenring-2025-12-31-softbanks-ai-vertical-play-integrating-ampere-and-graphcore-to-challenge-the-gpu-giants)
- [Esperanto Exit](https://xpu.pub/2025/07/05/esperanto/)

### FPGA
- [AMD Alveo V80](https://www.amd.com/en/products/accelerators/alveo/v80.html)
- [Intel Stratix 10 NX](https://www.hpcwire.com/2020/06/18/intel-debuts-stratix-10-nx-fpgas-targeting-ai-workloads/)

### Multi-GPU / Appliances
- [RTX 5090 LLM Benchmarks (RunPod)](https://www.runpod.io/blog/rtx-5090-llm-benchmarks)
- [NVIDIA DGX Spark](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/)
- [DGX Spark Alternatives](https://research.aimultiple.com/dgx-spark-alternatives/)
- [DGX Spark and Station Blog](https://blogs.nvidia.com/blog/dgx-spark-and-station-open-source-frontier-models/)
- [Local AI Hardware Landscape](https://www.starryhope.com/ai/dgx-spark-local-ai-hardware-landscape/)
- [Small Format AI Platforms](https://www.vset3d.com/small-format-ai-platforms-and-gpus-for-local-computation-in-2025/)

### Photonic / Edge
- [Lightmatter M1000](https://lightmatter.co/press-release/lightmatter-unveils-passage-m1000-photonic-superchip-worlds-fastest-ai-interconnect/)
- [Hailo AI Accelerators](https://hailo.ai/products/ai-accelerators/)
