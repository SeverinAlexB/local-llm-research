# AI Inference Hardware — Decision Summary

> Compiled February 16, 2026. Updated with MoE model analysis. All prices in CHF unless noted.

---

## Executive Summary

**The 60-80 tok/s target is achievable within budget — but not with dense 70B models.** The key insight: frontier coding models are rapidly moving to **Mixture of Experts (MoE)** architectures, where only a fraction of parameters are active per token. This means big-model quality at small-model speed. A single RTX PRO 6000 runs MoE coding models at **134-185 tok/s** — far exceeding the original target.

**Two critical hardware/model discoveries:**

1. **NVIDIA RTX PRO 6000 Blackwell** — a single card with 96 GB GDDR7, 1,792 GB/s bandwidth. Available in Switzerland from CHF 7,691 ([toppreise.ch](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823)). Dense 70B models: ~32 tok/s. MoE models: 134-185 tok/s.

2. **MoE coding models** (MiniMax-M2.5, GPT-OSS-120B, Qwen3-Coder-30B-A3B) deliver frontier-quality code generation while only activating 3-10B parameters per token, making them dramatically faster than dense 70B models on the same hardware.

---

## The MoE Advantage — Why This Changes Everything

Since early 2025, nearly all frontier coding models use **Mixture of Experts (MoE)** architecture. Unlike dense models (where every parameter is read for every token), MoE models only activate a small subset of "expert" parameters per token. This means:

- **Speed**: Only active parameters are read from VRAM → dramatically faster inference
- **Quality**: Total parameter count is large → model quality rivals or exceeds dense 70B models
- **VRAM**: Full model weights must still be stored → needs more VRAM than active params suggest

### MoE Coding Models That Fit on 96 GB (Single RTX PRO 6000)

| Model | Total Params | Active Params | Q4 Size | tok/s (PRO 6000) |
|---|---|---|---|---|
| **GPT-OSS-120B** | 120B | ~20B | ~65 GB | **134** |
| **GPT-OSS-20B** | 20B | ~5B | ~14 GB | **185** |
| **Qwen3-Coder-30B-A3B** | 30B | 3.3B | ~18 GB | **~150-200** (est.) |
| **Qwen3-30B-A3B** | 30B | 3.3B | ~18 GB | **~150-200** (est.) |
| **DeepSeek-Coder-V2-Lite** | 16B | 2.4B | ~10 GB | **~200+** (est.) |

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

**Key takeaway:** A single RTX PRO 6000 running GPT-OSS-120B at 134 tok/s delivers better coding quality AND 4x the speed of a dense 70B model at 32 tok/s.

Sources: [DatabaseMart PRO 6000 Benchmark](https://www.databasemart.com/blog/ollama-gpu-benchmark-pro6000), [NVIDIA MoE Blog](https://blogs.nvidia.com/blog/mixture-of-experts-frontier-models/), [Friendli MoE Comparison](https://friendli.ai/blog/moe-models-comparison), [Qwen3-30B-A3B HuggingFace](https://huggingface.co/Qwen/Qwen3-30B-A3B)

---

## Top 4 Configurations (Ranked)

> Full parts lists, shop links, and component pricing are in [build-guides.md](build-guides.md).

### #1 — Single RTX PRO 6000 Build (RECOMMENDED)

**Best value. MoE models make a single card sufficient. Leaves budget for upgrades.**

| Metric | Value |
|---|---|
| **Total cost** | ~CHF 9,500 (AM5) / ~CHF 10,700 (Threadripper) |
| **Total VRAM** | 96 GB |
| **MoE main agent** (GPT-OSS-120B Q4) | **~134 tok/s** — exceeds 60-80 target |
| **MoE sub-agents** (Qwen3-30B-A3B Q4) | **~150-200 tok/s** — far exceeds target |
| **Dense 70B fallback** (Qwen2.5-Coder 72B Q4) | ~32 tok/s — meets minimum |
| **Dense 32B speed** | ~56-64 tok/s |
| **Power** | ~400W under load |
| **Noise** | Quiet (single blower card, low system power) |
| **Upgrade path** | Swap to Threadripper + add 2nd PRO 6000 later for 192 GB |

**Why this is #1:** With MoE coding models, a single PRO 6000 delivers 134+ tok/s on the main agent — far exceeding the original target. The second GPU is no longer needed for speed. You save ~CHF 1,200-3,250 vs the dual-GPU builds.

**Concurrent MoE model loading on 96 GB:**

```
RTX PRO 6000 (96 GB):
├── Main agent:   GPT-OSS-120B Q4 (~65 GB) → ~134 tok/s
├── Sub-agents:   Qwen3-Coder-30B-A3B Q4 (~18 GB) → ~150-200 tok/s
├── Vision:       Qwen2.5-VL-7B Q4 (~5 GB)
└── Total loaded: ~88 GB, ~8 GB free for KV cache
```

**Tradeoff:** Concurrent heavy inference shares 1,792 GB/s bandwidth across all models. With the MoE layout at full concurrency (main + 5 sub-agents), expect ~80-100 tok/s main + ~30-50 tok/s per sub-agent. Still exceeds all targets.

---

### #2 — RTX PRO 6000 + RTX 5090 Build (Maximum Capability)

**For running the largest MoE models (MiniMax-M2.5) or dedicated GPUs for main vs sub-agents.**

| Metric | Value |
|---|---|
| **Total cost** | ~CHF 11,450 |
| **Total VRAM** | 128 GB (96 + 32) |
| **MoE main agent** (GPT-OSS-120B on PRO 6000) | **~134 tok/s** |
| **Sub-agents** (Qwen3-30B-A3B on RTX 5090) | **~150-200 tok/s** |
| **Concurrent strategy** | No bandwidth contention — models on separate GPUs |
| **Power draw** | ~875W under full load |
| **Noise** | Moderate (PRO 6000 blower + RTX 5090 open-air) |
| **Upgrade path** | Replace RTX 5090 with 2nd PRO 6000 for 192 GB |

**When this is worth the extra over Config #1:**
- You need **zero bandwidth contention** between main agent and sub-agents (dedicated GPUs)
- You want to run **MiniMax-M2.5 NVFP4** (~130 GB) spanning both GPUs
- You want **dense 70B on PRO 6000 + concurrent sub-agents on RTX 5090** without sharing bandwidth
- Future-proofing: 128 GB total vs 96 GB

---

### #2b — DGX Spark Cluster (Silent, Turnkey Alternative)

**Cheapest path to the 60-80 tok/s target. Near-silent. No build required.**

| Metric | Value |
|---|---|
| **Total cost** | ~CHF 5,000 (2× ASUS Ascent) / ~CHF 7,500 (3× ASUS Ascent) |
| **Total memory** | 256 GB (2 units) / 384 GB (3 units) |
| **MoE main agent** (GPT-OSS-120B, TP=2) | **~75 tok/s** — meets 60-80 target |
| **MoE sub-agent** (3rd unit, standalone) | **~50-55 tok/s** |
| **Dense 70B fallback** | ~5 tok/s — **unusable** |
| **Power** | ~280W (2 units) / ~420W (3 units) |
| **Noise** | **Near silent** |

**Tradeoff vs PRO 6000:** 1.8× slower on MoE models (75 vs 134 tok/s). Dense 70B is a dead path (~5 tok/s). But half the price, near-silent, turnkey, and massive memory (256-384 GB).

---

### #3 — Mac Studio M3 Ultra 256 GB (Silent Option)

**Best for noise-sensitivity. Massive memory. Slowest on 70B.**

| Metric | Value |
|---|---|
| **Total cost** | ~CHF 6,100 |
| **Total memory** | 256 GB unified (exceeds target) |
| **MoE main agent** (GPT-OSS-120B Q4) | **~69 tok/s** — meets 60-80 target |
| **MoE sub-agents** (Qwen3-30B-A3B Q4) | **~84 tok/s** single / ~25 tok/s at 8 concurrent |
| **Dense 70B Q4 fallback** | ~15-25 tok/s (MLX) — below 30 tok/s minimum |
| **Memory bandwidth** | 819 GB/s (all shared) |
| **Power** | ~200-300W |
| **Noise** | **Silent** |

**Why #3 and not higher:** The PRO 6000 is nearly 2x faster on MoE models (134 vs 69 tok/s) and degrades less under concurrency. But the Mac Studio's 256 GB memory, silent operation, zero build effort, and lower price make it a strong alternative — especially if you value silence and memory headroom over raw speed.

Sources: [Apple CH Store](https://www.apple.com/ch-de/shop/buy-mac/mac-studio), [Creative Strategies Review](https://creativestrategies.com/mac-studio-m3-ultra-ai-workstation-review/), [Olares Blog Benchmarks](https://blog.olares.com/local-ai-hardware-performance-benchmarking/), [vllm-mlx Paper](https://arxiv.org/html/2601.19139v1)

---

### #4 — 2x RTX 5090 Build (Speed-Focused, VRAM-Limited)

**Fastest per-stream for small/medium models. Insufficient VRAM for concurrent 70B + sub-agents.**

| Metric | Value |
|---|---|
| **Total cost** | ~CHF 7,665 |
| **Total VRAM** | 64 GB (well below 128 target) |
| **70B Q4 speed** (2 GPUs) | ~27-40 tok/s |
| **32B Q4 speed** (1 GPU) | ~61 tok/s |
| **Concurrent models** | TIGHT — 70B Q4 (~43 GB) fills both GPUs; ~21 GB left for sub-agents |
| **Power** | ~1,350W under full load |
| **Noise** | Moderate-loud (2x 575W GPUs) |

**Why it's ranked #4:** The 64 GB VRAM is a hard constraint. Running a 70B Q4 model takes ~43 GB, leaving only ~21 GB for everything else. For a multi-agent workflow, this is too restrictive.

---

## Configurations NOT Recommended

| Config | Why Not |
|---|---|
| **4x RTX A6000 (used)** | 192 GB VRAM is great, but ~20-30 tok/s on 70B (low bandwidth), extremely loud blower coolers, ~CHF 13,600+ |
| **4x RTX 4090** | 96 GB, ~CHF 14,200, extremely loud, 2,000W power, stock scarce |
| **2x A100 80GB (used)** | Good bandwidth but passive cooling (datacenter card), ~CHF 12,000+ |
| **Mac Pro M2 Ultra** | Older chip, max 192 GB, more expensive than M3 Ultra Mac Studio, Apple has abandoned the product line |
| **Pre-built from BIZON** | Good quality but CHF 14,000-16,500 for 2x RTX 5090 (only 64 GB VRAM) — over budget for equivalent specs |
| **4x RTX 3090 (used)** | 96 GB, great value at ~CHF 5,500, but ~15-22 tok/s on 70B — too slow |

---

## Decision Framework

```
Best overall (with MoE models)?
  → Single RTX PRO 6000 (~CHF 9,500)
    Get: 96 GB, MoE main agent at 134 tok/s, MoE sub-agents at 150-200 tok/s
    Upgrade path: add 2nd GPU later

Need zero bandwidth contention or largest MoE models (MiniMax-M2.5)?
  → RTX PRO 6000 + RTX 5090 (~CHF 11,450)
    Get: 128 GB, dedicated GPUs for main vs sub-agents

Prioritize BUDGET + SILENCE above all?
  → 2x DGX Spark / ASUS Ascent (~CHF 5,000)
    Get: MoE main agent at ~75 tok/s, 256 GB memory, turnkey, near-silent
    Accept: dense 70B unusable (~5 tok/s), only 2 concurrent streams

Prioritize SILENCE + MEMORY above all?
  → Mac Studio M3 Ultra 256 GB (~CHF 6,100)
    Get: MoE main agent at ~69 tok/s, 256 GB memory, silent
    Accept: ~half the MoE speed of PRO 6000, concurrent performance degrades faster

Want MAX SPEED on dense models only?
  → 2x RTX 5090 (~CHF 7,665)
    Get: 32B at ~61 tok/s, but only 64 GB VRAM — limits concurrent models
```

---

## Detailed Research Files

| File | What's inside |
|------|---------------|
| [build-guides.md](build-guides.md) | Full parts lists, shop links, and component pricing for each config |
| [benchmarks.md](benchmarks.md) | All benchmark tables, memory requirements, inference software comparison |
| [gpu-research-nvidia.md](gpu-research-nvidia.md) | Deep-dive on every NVIDIA GPU considered |
| [apple-silicon-research.md](apple-silicon-research.md) | Mac Studio M3 Ultra / M4 Max analysis |
| [vendor-options.md](vendor-options.md) | Pre-built workstation vendors |
| [requirements.md](requirements.md) | Hardware requirements |
