# AI Inference Hardware Research

> **Budget:** CHF 4,000–15,000 | **Date:** February 2026 | **Use case:** Local multi-agent coding inference (home office, headless server)

Research into building a local AI inference server for multi-agent coding workflows. The key finding: **MoE (Mixture of Experts) coding models** changed everything — a single NVIDIA RTX PRO 6000 Blackwell (96 GB, CHF ~9,500 full build) runs frontier coding models at **134 tok/s**, far exceeding the 60-80 tok/s target.

## Bottom Line

1. **Best overall:** Single RTX PRO 6000 build (~CHF 9,500) — 134 tok/s MoE, 96 GB, quiet, upgradeable
2. **Silent alternative:** Mac Studio M3 Ultra 256 GB (~CHF 6,100) — 69 tok/s MoE, zero build effort
3. **Budget cluster:** 2× DGX Spark / ASUS Ascent (~CHF 5,000) — 75 tok/s MoE, turnkey, 256 GB

## Why Only Three Platforms?

We surveyed [30+ hardware platforms](alternative-platforms-research.md) across the entire AI accelerator landscape. Local LLM inference needs two things simultaneously: **large memory** (96 GB+ to fit frontier models) and **high memory bandwidth** (to generate tokens fast). Most platforms deliver one but not the other — or aren't available to buy at all.

**Datacenter chips are inaccessible.** AMD Instinct, Intel Gaudi, Cerebras, Groq, Google TPU, AWS Trainium — all enterprise-only ($15K–millions), passive-cooled, not sold as individual cards. Chinese chips (Huawei Ascend, Biren) are also export-restricted.

**Buyable alternatives all compromise.** AMD W7900 has bandwidth but only 48 GB. Strix Halo has 128 GB but only 256 GB/s (~7 tok/s). Tenstorrent Blackhole is $1,299 but its software stack is pre-production. Intel Arc Pro B60 has only 24 GB.

**Software seals it.** CUDA (vLLM, llama.cpp, Ollama) is years ahead. ROCm and oneAPI still add friction. Apple's MLX is the only non-CUDA stack that's production-ready.

This leaves three viable paths: **NVIDIA discrete GPUs** (best bandwidth + CUDA), **Apple Silicon** (best unified memory + MLX), and **NVIDIA DGX Spark / AMD Strix Halo** (largest memory pool at lowest cost, accepting slow bandwidth).

## Files

| File | Purpose |
|------|---------|
| [requirements.md](requirements.md) | What we need — performance targets, budget, constraints |
| [decision-summary.md](decision-summary.md) | Which configs meet those requirements and why |
| [build-guides.md](build-guides.md) | Exact parts lists, prices, shop links for each build |
| [benchmarks.md](benchmarks.md) | All performance data — tok/s tables, memory requirements, software comparison |
| [gpu-research-nvidia.md](gpu-research-nvidia.md) | Deep-dive on every NVIDIA GPU considered |
| [apple-silicon-research.md](apple-silicon-research.md) | Mac Studio M3 Ultra / M4 Max analysis |
| [vendor-options.md](vendor-options.md) | Pre-built workstation vendors (BIZON, BRESSNER, RECT) |
| [alternative-platforms-research.md](alternative-platforms-research.md) | Full survey of 30+ competing platforms (AMD, Intel, Tenstorrent, startups, etc.) |

## Reading Order

**For the purchase decision:** requirements → decision-summary → build-guides. Done.

**For deep-dives:** benchmarks (all perf data), gpu-research-nvidia (GPU details), apple-silicon-research (Mac path), vendor-options (pre-built alternatives), alternative-platforms-research (why no one else competes).
