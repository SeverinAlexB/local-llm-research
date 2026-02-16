# AI Inference Hardware — Research Report

> Compiled February 16, 2026. Updated with MoE model analysis. All prices in CHF unless noted. Benchmark numbers are approximate and depend on model, quantization, context length, and inference engine.

---

## Executive Summary

**The 60-80 tok/s target is achievable within budget — but not with dense 70B models.** The key insight: frontier coding models are rapidly moving to **Mixture of Experts (MoE)** architectures, where only a fraction of parameters are active per token. This means big-model quality at small-model speed. A single RTX PRO 6000 runs MoE coding models at **134-185 tok/s** — far exceeding the original target.

**Two critical hardware/model discoveries:**

1. **NVIDIA RTX PRO 6000 Blackwell** — a single card with 96 GB GDDR7, 1,792 GB/s bandwidth. Available in Switzerland from CHF 6,443. Dense 70B models: ~32 tok/s. MoE models: 134-185 tok/s.

2. **MoE coding models** (MiniMax-M2.5, GPT-OSS-120B, Qwen3-Coder-30B-A3B) deliver frontier-quality code generation while only activating 3-10B parameters per token, making them dramatically faster than dense 70B models on the same hardware.

---

## The MoE Advantage — Why This Changes Everything

Since early 2025, nearly all frontier coding models use **Mixture of Experts (MoE)** architecture. Unlike dense models (where every parameter is read for every token), MoE models only activate a small subset of "expert" parameters per token. This means:

- **Speed**: Only active parameters are read from VRAM → dramatically faster inference
- **Quality**: Total parameter count is large → model quality rivals or exceeds dense 70B models
- **VRAM**: Full model weights must still be stored → needs more VRAM than active params suggest

### MoE Coding Models That Fit on 96 GB (Single RTX PRO 6000)

| Model | Total Params | Active Params | Q4 Size | Fits 96GB? | tok/s (PRO 6000) |
|---|---|---|---|---|---|
| **GPT-OSS-120B** | 120B | ~20B | ~65 GB | **Yes** | **134** |
| **GPT-OSS-20B** | 20B | ~5B | ~14 GB | **Yes** | **185** |
| **Qwen3-Coder-30B-A3B** | 30B | 3.3B | ~18 GB | **Yes** | **~150-200** (est.) |
| **Qwen3-30B-A3B** | 30B | 3.3B | ~18 GB | **Yes** | **~150-200** (est.) |
| **DeepSeek-Coder-V2-Lite** | 16B | 2.4B | ~10 GB | **Yes** | **~200+** (est.) |

### MoE Models That Do NOT Fit on 96 GB

| Model | Total Params | Active Params | Q4 Size | Minimum VRAM |
|---|---|---|---|---|
| **MiniMax-M2.5** | 230B | 10B | ~130 GB | 2x PRO 6000 (192 GB) |
| **DeepSeek-V3/R1** (full) | 671B | 37B | ~386 GB | 5+ GPUs |

### Dense Models for Comparison

| Model | Params | Active | Q4 Size | tok/s (PRO 6000) |
|---|---|---|---|---|
| Qwen2.5-Coder 72B | 72B | 72B (all) | ~36 GB | **29-32** |
| DeepSeek-R1 Distill 32B | 32B | 32B (all) | ~20 GB | **64** |
| LLaMA 3.3 70B | 70B | 70B (all) | ~34 GB | **32** |

**Key takeaway:** A single RTX PRO 6000 running GPT-OSS-120B at 134 tok/s delivers better coding quality AND 4x the speed of a dense 70B model at 32 tok/s. MoE models make the original 60-80 tok/s target easily achievable — and then some.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [NVIDIA MoE Blog](https://blogs.nvidia.com/blog/mixture-of-experts-frontier-models/), [Friendli MoE Comparison](https://friendli.ai/blog/moe-models-comparison), [Qwen3-30B-A3B HuggingFace](https://huggingface.co/Qwen/Qwen3-30B-A3B)

---

## Top 4 Configurations (Ranked)

### #1 — Single RTX PRO 6000 Build (RECOMMENDED)

**Best value. MoE models make a single card sufficient. Leaves budget for upgrades.**

| Component | Spec | CHF |
|---|---|---|
| GPU | NVIDIA RTX PRO 6000 Blackwell (96 GB) | 6,443 |
| CPU | AMD Ryzen 9 9950X (16C) | 550 |
| Motherboard | AM5 X870E (PCIe 5.0 x16) | 350 |
| RAM | 64 GB DDR5-5600 (2x32 GB) | 160 |
| PSU | be quiet! Pure Power 12 M 1000W | 180 |
| Case | Fractal Design Define 7 | 150 |
| CPU Cooler | Noctua NH-D15 | 100 |
| Storage | Samsung 990 Pro 2 TB NVMe | 200 |
| Misc | Cables | 50 |
| **Total** | | **~CHF 8,200** |

| Metric | Value |
|---|---|
| **Total VRAM** | 96 GB |
| **MoE main agent** (GPT-OSS-120B Q4) | **~134 tok/s** — exceeds 60-80 target |
| **MoE sub-agents** (Qwen3-30B-A3B Q4) | **~150-200 tok/s** — far exceeds target |
| **Dense 70B fallback** (Qwen2.5-Coder 72B Q4) | ~32 tok/s — meets minimum |
| **Dense 32B speed** | ~56-64 tok/s |
| **Power** | ~400W under load |
| **Noise** | Quiet (single blower card, low system power) |
| **Upgrade path** | Swap to Threadripper + add 2nd PRO 6000 later for 192 GB |

**Why this is now #1:** With MoE coding models, a single PRO 6000 delivers 134+ tok/s on the main agent — far exceeding the original target. The second GPU (RTX 5090) is no longer needed for speed. You save ~CHF 3,250 vs the dual-GPU build.

**Concurrent MoE model loading on 96 GB:**

```
RTX PRO 6000 (96 GB):
├── Main agent:   GPT-OSS-120B Q4 (~65 GB) → ~134 tok/s
├── Sub-agents:   Qwen3-Coder-30B-A3B Q4 (~18 GB) → ~150-200 tok/s
├── Vision:       Qwen2.5-VL-7B Q4 (~5 GB)
└── Total loaded: ~88 GB, ~8 GB free for KV cache
```

Alternative layout with dense fallback:

```
RTX PRO 6000 (96 GB):
├── Complex tasks: Qwen2.5-Coder 72B Q4 (~36 GB) → ~32 tok/s
├── Fast agent:    Qwen3-Coder-30B-A3B Q4 (~18 GB) → ~150-200 tok/s
├── Vision:        Qwen2.5-VL-7B Q4 (~5 GB)
└── Total loaded: ~59 GB, ~37 GB free for KV cache
```

**Tradeoff:** Concurrent heavy inference shares 1,792 GB/s bandwidth across all models. With the MoE layout at full concurrency (main + 5 sub-agents), expect ~80-100 tok/s main + ~30-50 tok/s per sub-agent. Still exceeds all targets.

**Key benchmark data for RTX PRO 6000 Blackwell (single card, Ollama):**

| Model | Type | Params (total/active) | Quant | tok/s |
|---|---|---|---|---|
| GPT-OSS-120B | MoE | 120B / ~20B | Q4 | **134** |
| GPT-OSS-20B | MoE | 20B / ~5B | Q4 | **185** |
| DeepSeek-R1 | Dense | 32B / 32B | Q4 | 64 |
| Qwen 3 | Dense | 32B / 32B | Q4 | 56 |
| Gemma 3 | Dense | 27B / 27B | Q4 | 62 |
| LLaMA 3.3 | Dense | 70B / 70B | Q4 | 32 |
| DeepSeek-R1 | Dense | 70B / 70B | Q4 | 32 |
| Qwen 2.5 | Dense | 72B / 72B | Q4 | 29 |

Sources: [DatabaseMart Ollama Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [CloudRift GPU Comparison](https://www.cloudrift.ai/blog/benchmarking-rtx-gpus-for-llm-inference)

---

### #2 — RTX PRO 6000 + RTX 5090 Build (Maximum Capability)

**For running the largest MoE models (MiniMax-M2.5) or dense 70B + sub-agents on separate GPUs.**

| Component | Spec | CHF |
|---|---|---|
| GPU 1 | NVIDIA RTX PRO 6000 Blackwell (96 GB) | 6,443 |
| GPU 2 | NVIDIA RTX 5090 (32 GB) | 2,080 |
| CPU | AMD Threadripper 7960X (24C) | 1,025 |
| Motherboard | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| RAM | 128 GB DDR5-5600 (4x32 GB) | 300 |
| PSU | be quiet! Dark Power Pro 13 1600W | 370 |
| Case | Fractal Design Define 7 XL | 165 |
| CPU Cooler | Noctua NH-U14S TR5-SP6 | 120 |
| Storage | Samsung 990 Pro 2 TB NVMe | 200 |
| Misc | Cables, fans | 100 |
| **Total** | | **~CHF 11,450** |

| Metric | Value |
|---|---|
| **Total VRAM** | 128 GB (meets target) |
| **MoE main agent** (GPT-OSS-120B on PRO 6000) | **~134 tok/s** |
| **Dense 70B** (PRO 6000) | ~32 tok/s |
| **Sub-agents** (Qwen3-30B-A3B on RTX 5090) | **~150-200 tok/s** |
| **Concurrent strategy** | No bandwidth contention — models on separate GPUs |
| **Power draw** | ~875W under full load |
| **Noise** | Moderate (PRO 6000 blower + RTX 5090 open-air) |
| **Upgrade path** | Replace RTX 5090 with 2nd PRO 6000 for 192 GB |

**When this is worth the extra CHF 3,250 over Config #1:**
- You need **zero bandwidth contention** between main agent and sub-agents (dedicated GPUs)
- You want to run **MiniMax-M2.5 NVFP4** (~130 GB) spanning both GPUs at ~83 tok/s with 32 concurrent users
- You want **dense 70B on PRO 6000 + concurrent sub-agents on RTX 5090** without sharing bandwidth
- Future-proofing: 128 GB total vs 96 GB

---

### #3 — Mac Studio M3 Ultra 256 GB (Silent Option)

**Best for noise-sensitivity. Massive memory. Slowest on 70B.**

| Component | Spec | CHF |
|---|---|---|
| Mac Studio | M3 Ultra 28C/60C, 256 GB, 2 TB | ~6,100 |
| **Total** | | **~CHF 6,100** |

| Metric | Value |
|---|---|
| **Total memory** | 256 GB unified (exceeds target) |
| **MoE main agent** (GPT-OSS-120B Q4) | **~69 tok/s** — meets 60-80 target |
| **MoE sub-agents** (Qwen3-30B-A3B Q4) | **~84 tok/s** single / ~25 tok/s at 8 concurrent |
| **Dense 70B Q4 fallback** | ~15-25 tok/s (MLX) — below 30 tok/s minimum |
| **Dense 32B Q4 speed** | ~33-41 tok/s |
| **Memory bandwidth** | 819 GB/s (all shared) |
| **Power** | ~200-300W |
| **Noise** | **Silent** |
| **Form factor** | Small box, turnkey, zero build effort |

**The practical Apple Silicon strategy:**
- Use **GPT-OSS-120B Q4** (MoE) as the main agent (~69 tok/s) — meets 60-80 tok/s target
- Use **Qwen3-Coder-30B-A3B Q4** (MoE) for sub-agents (~84 tok/s single, ~25 tok/s at 8 concurrent)
- Use **dense 70B models** for complex reasoning tasks where you can tolerate ~15-20 tok/s
- All models loaded concurrently — 256 GB is more than enough

**Limitations:**
- MoE main agent at ~69 tok/s is about half the PRO 6000's ~134 tok/s
- Dense 70B at ~15-20 tok/s falls short of 30 tok/s minimum (use MoE models instead)
- Concurrent MoE performance degrades faster: GPT-OSS-120B drops from 69 to ~19 tok/s at 8 concurrent streams
- No CUDA — limited to MLX/Metal ecosystem (vllm-mlx, Ollama, llama.cpp Metal)
- Cannot upgrade or expand later
- No M4 Ultra exists — M3 Ultra is the latest available
- Image generation is 2-8x slower than NVIDIA (irrelevant since you'll use cloud APIs)

**Software:** Use **vllm-mlx** for concurrent serving (continuous batching, OpenAI-compatible API, vision model support). It achieves 21-87% higher throughput than llama.cpp.

**MoE performance on M3 Ultra** (from Olares benchmark suite):

| Model | Type | 1 concurrent | 8 concurrent |
|---|---|---|---|
| GPT-OSS-120B | MoE (120B/~20B active) | **69.4 tok/s** | 19.1 tok/s |
| Qwen3-30B-A3B | MoE (30B/3.3B active) | **84 tok/s** | 24.9 tok/s |

**Why this is #3 and not higher:** The PRO 6000 is nearly 2x faster on MoE models (134 vs 69 tok/s) and degrades less under concurrency. But the Mac Studio's 256 GB memory, silent operation, zero build effort, and lower price (~CHF 6,100 vs ~CHF 8,200) make it a strong alternative — especially if you value silence and memory headroom over raw speed.

Sources: [Apple CH Store](https://www.apple.com/ch-de/shop/buy-mac/mac-studio), [Creative Strategies Review](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/), [Olares Blog Benchmarks](https://blog.olares.com/local-ai-hardware-performance-benchmarking/), [vllm-mlx Paper](https://arxiv.org/html/2601.19139v1)

---

### #4 — 2x RTX 5090 Build (Speed-Focused, VRAM-Limited)

**Fastest per-stream for small/medium models. Insufficient VRAM for concurrent 70B + sub-agents.**

| Component | Spec | CHF |
|---|---|---|
| GPU | 2x NVIDIA RTX 5090 (32 GB each) | 4,160 |
| CPU | AMD Threadripper 7960X (24C) | 1,025 |
| Motherboard | ASUS Pro WS TRX50-SAGE WIFI | 650 |
| RAM | 128 GB DDR5-5600 (4x32 GB) | 300 |
| PSU | be quiet! Dark Power Pro 13 1600W | 370 |
| Case | Fractal Design Define 7 XL | 165 |
| CPU Cooler | Noctua NH-U14S TR5-SP6 | 120 |
| Storage | Samsung 990 Pro 2 TB NVMe | 200 |
| Misc | Cables, fans | 100 |
| **Total** | | **~CHF 7,090** |

| Metric | Value |
|---|---|
| **Total VRAM** | 64 GB (well below 128 target) |
| **70B Q4 speed** (2 GPUs) | ~27-40 tok/s |
| **32B Q4 speed** (1 GPU) | ~61 tok/s |
| **7B sub-agents** (1 GPU) | ~200+ tok/s |
| **Concurrent models** | TIGHT — 70B Q4 (~43 GB) fills both GPUs; ~21 GB left for sub-agents |
| **Power** | ~1,350W under full load |
| **Noise** | Moderate-loud (2x 575W GPUs) |

**Why it's ranked #4:** The 64 GB VRAM is a hard constraint. Running a 70B Q4 model takes ~43 GB, leaving only ~21 GB for everything else. You can't comfortably run 70B + 32B sub-agent + vision concurrently. You'd need to swap models or use only 7B sub-agents. For a multi-agent workflow, this is too restrictive.

---

## Configurations NOT Recommended

| Config | Why Not |
|---|---|
| **4x RTX A6000 (used)** | 192 GB VRAM is great, but ~20-30 tok/s on 70B (low bandwidth 768 GB/s), extremely loud blower coolers, ~CHF 13,600+ with refurb risk |
| **4x RTX 4090** | 96 GB, ~CHF 14,200, extremely loud, 2,000W power draw, physically hard to fit, stock scarce |
| **2x A100 80GB (used)** | Good bandwidth but passive cooling (datacenter card), ~CHF 12,000+, noise requires custom solution |
| **Mac Pro M2 Ultra** | Older chip, max 192 GB, more expensive than M3 Ultra Mac Studio, Apple has abandoned the product line |
| **Pre-built from BIZON** | Good quality but CHF 14,000-16,500 for 2x RTX 5090 (only 64 GB VRAM) — over budget for equivalent specs |
| **4x RTX 3090 (used)** | 96 GB, great value at ~CHF 5,500, but ~15-22 tok/s on 70B — too slow |

---

## Key Technical Details

### Memory Requirements for Target Models

**MoE Models:**

| Model | Total / Active Params | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|---|
| GPT-OSS-120B | 120B / ~20B | Q4 | ~65 GB | ~2 GB | ~67 GB |
| Qwen3-Coder-30B-A3B | 30B / 3.3B | Q4 | ~18 GB | ~1 GB | ~19 GB |
| MiniMax-M2.5 | 230B / 10B | NVFP4 | ~130 GB | ~5 GB | ~135 GB |
| DeepSeek-Coder-V2-Lite | 16B / 2.4B | Q4 | ~10 GB | ~1 GB | ~11 GB |

**Dense Models:**

| Model | Quant | Weights | + KV Cache (8K) | Total |
|---|---|---|---|---|
| Qwen2.5-Coder 72B | Q4 | ~36 GB | ~2.5 GB | ~39 GB |
| Qwen2.5-Coder 72B | FP8 | ~72 GB | ~2.5 GB | ~75 GB |
| Qwen2.5-Coder 32B | Q4 | ~18 GB | ~1.5 GB | ~20 GB |
| Llama 3.1 70B | Q4 | ~34 GB | ~2.5 GB | ~37 GB |
| DeepSeek-R1 Distill 32B | Q4 | ~20 GB | ~1.5 GB | ~22 GB |
| Qwen2.5-VL-7B (vision) | Q4 | ~4 GB | ~1 GB | ~5 GB |

### Inference Software Recommendation

**For NVIDIA multi-GPU:** Use **vLLM** or **SGLang** — they support tensor parallelism and continuous batching. SGLang maintains more consistent per-request speeds under load (30-31 tok/s stable vs vLLM's degradation).

**For Apple Silicon:** Use **vllm-mlx** — brings continuous batching to MLX with OpenAI-compatible API. 21-87% faster than llama.cpp.

**For quick setup / development:** **Ollama** is easiest. But for production multi-agent serving, switch to vLLM/SGLang.

| Feature | vLLM | SGLang | Ollama | llama.cpp |
|---|---|---|---|---|
| Multi-GPU tensor parallel | Yes | Yes | No | No (layer split only) |
| Continuous batching | Yes | Yes | Limited | Limited (slots) |
| Throughput at scale | Very High | Very High | Low | Low |
| Setup complexity | Medium | Medium | Very Easy | Medium |
| Apple Silicon | Via vllm-mlx | No | Yes | Yes (Metal) |

### Concurrent Inference Strategy

**Config #1 — Single PRO 6000 with MoE models (recommended):**

```
RTX PRO 6000 (96 GB):
├── vLLM/SGLang: GPT-OSS-120B Q4 (~65 GB) — main agent @ ~134 tok/s
├── vLLM/SGLang: Qwen3-Coder-30B-A3B Q4 (~18 GB) — sub-agents @ ~150-200 tok/s
├── vLLM/SGLang: Qwen2.5-VL-7B Q4 (~5 GB) — vision/QA
└── ~8 GB free for KV cache

Routing: LiteLLM or nginx reverse proxy in front
```

**Config #2 — PRO 6000 + RTX 5090 with GPU separation:**

```
GPU 0 (PRO 6000, 96 GB):
├── GPT-OSS-120B Q4 (~65 GB) or Qwen2.5-Coder 72B Q4 (~39 GB) — main agent
├── Qwen2.5-VL-7B Q4 (~5 GB) — vision/QA
└── Free VRAM for KV cache

GPU 1 (RTX 5090, 32 GB):
├── Qwen3-Coder-30B-A3B Q4 (~18 GB) — sub-agents @ ~150-200 tok/s
└── ~14 GB free for KV cache

No bandwidth contention between main agent and sub-agents.
```

### RTX PRO 6000 Blackwell — Key Specs

| Spec | Value |
|---|---|
| Architecture | Blackwell (GB202) |
| CUDA Cores | 24,064 |
| Tensor Cores | 752 |
| VRAM | 96 GB GDDR7 ECC |
| Memory Bus | 512-bit |
| Memory Bandwidth | 1,792 GB/s |
| TDP | 300W |
| Cooling | Active blower (Workstation Edition) |
| NVLink | Supported (professional feature) |
| FP4 Support | Yes (1.32x throughput improvement over FP8) |
| PCIe | Gen 5.0 x16 |
| Price (CH) | CHF 6,443 (Workstation Ed.) / CHF 6,749 (Server Ed.) |

Note: The **Server Edition** is passively cooled (requires server chassis with forced airflow). For a home office tower, use the **Workstation Edition** with its active blower cooler.

Sources: [NVIDIA RTX PRO 6000 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/data-center/rtx-pro-6000-blackwell-workstation-edition/workstation-blackwell-rtx-pro-6000-workstation-edition-nvidia-us-3519208-web.pdf), [StorageReview](https://www.storagereview.com/review/nvidia-rtx-pro-6000-workstation-gpu-review-blackwell-architecture-and-96-gb-for-pro-workflows), [GamersNexus Benchmarks](https://gamersnexus.net/gpus/nvidia-rtx-pro-6000-blackwell-benchmarks-tear-down-thermals-gaming-llm-acoustic-tests)

---

## Swiss Retailer Availability (February 2026)

### RTX PRO 6000 Blackwell

| Variant | Price (CHF) | Source |
|---|---|---|
| Workstation Edition (NVIDIA) | from 6,443 | [Toppreise.ch](https://www.toppreise.ch) |
| Server Edition (NVIDIA) | from 6,749 | [Toppreise.ch](https://www.toppreise.ch) |
| PNY Workstation Edition | from 7,848 | [Toppreise.ch](https://www.toppreise.ch) |

### RTX 5090

| Model | Price (CHF) | Source |
|---|---|---|
| ZOTAC (cheapest) | from 2,080 | [Toppreise.ch](https://www.toppreise.ch) |
| ASUS | from 2,099 | Toppreise.ch |
| MSI | from 2,339 | Toppreise.ch |
| Premium models | 2,600-4,959 | Digitec.ch |

### Build Components

| Component | Specific Product | CHF | Source |
|---|---|---|---|
| CPU | AMD Threadripper 7960X | 1,025 | Toppreise.ch |
| Motherboard | ASUS Pro WS TRX50-SAGE WIFI | 649 | Toppreise.ch |
| RAM | Kingston FURY Beast 128GB DDR5-5600 | 300 | Toppreise.ch |
| PSU | be quiet! Dark Power Pro 13 1600W | 368 | Toppreise.ch |
| Case | Fractal Design Define 7 XL | 163 | Toppreise.ch |
| CPU Cooler | Noctua NH-U14S TR5-SP6 | 120 | Toppreise.ch |
| Storage | Samsung 990 Pro 2TB | 195 | Toppreise.ch |

### Mac Studio

| Configuration | CHF | Source |
|---|---|---|
| M3 Ultra 28C/60C, 96 GB, 1 TB | 4,199 | Apple CH |
| M3 Ultra 28C/60C, 256 GB, 2 TB | ~6,100 | Apple CH (CTO) |
| M3 Ultra 32C/80C, 512 GB, 2 TB | ~8,400 | Apple CH (CTO) |

---

## Decision Framework

```
Best overall (with MoE models)?
  → Single RTX PRO 6000 (~CHF 8,200)
    Get: 96 GB, MoE main agent at 134 tok/s, MoE sub-agents at 150-200 tok/s
    Upgrade path: add 2nd GPU later

Need zero bandwidth contention or largest MoE models (MiniMax-M2.5)?
  → RTX PRO 6000 + RTX 5090 (~CHF 11,450)
    Get: 128 GB, dedicated GPUs for main vs sub-agents

Prioritize SILENCE above all?
  → Mac Studio M3 Ultra 256 GB (~CHF 6,100)
    Get: MoE main agent at ~69 tok/s (GPT-OSS-120B), MoE sub-agents at ~84 tok/s, 256 GB memory
    Accept: ~half the MoE speed of PRO 6000, concurrent performance degrades faster

Want MAX SPEED on dense models only?
  → 2x RTX 5090 (~CHF 7,090)
    Get: 32B at ~61 tok/s, but only 64 GB VRAM — limits concurrent models
```

---

## Detailed Research Files

The following files contain full research data with all sources:

- `apple-silicon-research.md` — Mac Studio M3 Ultra / M4 Max deep-dive
- `GPU-RESEARCH-NVIDIA.md` — All NVIDIA GPU options analyzed
- `WORKSTATION-OPTIONS.md` — Pre-built vendors and DIY build configs
- `BENCHMARKS.md` — Comprehensive performance benchmarks and software comparison
- `REQUIREMENTS.md` — Your hardware requirements
