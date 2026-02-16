# AI Inference Hardware — Requirements

## Overview

Local AI inference hardware for home office use, primarily for AI-assisted coding with multi-agent workflows. Must support remote access (SSH/API from laptop — no local software needed).

## Use Cases (by priority)

1. **Code generation** — Primary workload. Frontier coding models for main/complex tasks, smaller models for sub-agents. The industry trend is toward **MoE (Mixture of Experts)** architectures (e.g., MiniMax-M2.5, GPT-OSS-120B, Qwen3-Coder-30B-A3B), which activate only a fraction of parameters per token (e.g., 10B active out of 230B total), delivering big-model quality at small-model speed. Dense models (e.g., Qwen2.5-Coder 72B, Llama 3.1 70B) remain relevant as fallbacks for specific tasks.
2. **Image recognition** — Vision models analyzing screenshots for automated QA (e.g., an agent takes a screenshot of an app and evaluates it). Runs concurrently with code generation.
3. **Image generation** — Not required. Cloud API fallback is acceptable.

## Performance Requirements

| Requirement | Target | Minimum |
|---|---|---|
| Main agent speed | 60-80 tok/s | 30 tok/s |
| Sub-agent speed | 15-20 tok/s | ~10 tok/s |
| Concurrent streams | 5-10 | 3 |
| Image recognition | Concurrent with LLM inference | — |

- Multi-agent pattern: one main agent (needs speed) + multiple sub-agents (can be slower, can use smaller models)
- All streams may run simultaneously alongside image recognition
- **MoE note**: The 60-80 tok/s target is achievable with MoE coding models (e.g., GPT-OSS-120B at ~134 tok/s on a single RTX PRO 6000). Dense 70B models max out at ~32 tok/s on the same hardware — MoE models are the practical path to meeting speed targets.

## Memory (VRAM)

- **Target**: 128-256 GB GPU-accessible RAM
- **Practical minimum**: 96 GB GPU-accessible RAM
- Must be GPU RAM (VRAM or unified memory) — not system RAM with CPU offloading
- Sufficient to hold a main coding model + sub-agent model(s) + a vision model, all loaded concurrently
- **MoE note**: MoE models store all parameters in VRAM but only activate a subset per token. A 120B MoE model in Q4 (~65 GB) + a 30B MoE sub-agent model (~18 GB) + a 7B vision model (~5 GB) = ~88 GB total — fits on a single 96 GB card. The original 256 GB target assumed dense models; with MoE, **96 GB is sufficient** for most frontier coding workloads. 128-192 GB provides headroom for the largest MoE models (e.g., MiniMax-M2.5 at ~130 GB in NVFP4).

## Quantization

- FP8 preferred for dense models
- FP4 / NVFP4 acceptable and increasingly relevant — Blackwell GPUs have native FP4 hardware support, and many MoE models are distributed in NVFP4 format (e.g., MiniMax-M2.5-NVFP4)
- Q4 (GGUF) is the practical default for most local inference via Ollama/llama.cpp
- Full FP16 not required

## Budget

- **Range**: 4,000 – 15,000 CHF (~4,400 – 16,500 USD)
- Sweet spot around 5,000 – 12,000 CHF if possible
- Open to refurbished/used hardware, but risk-conscious (concerned about receiving broken expensive components with limited recourse)

## Environment Constraints

- **Location**: Home office (not a server room)
- **Noise**: Must be reasonably quiet — no jet-engine server fans
- **Form factor**: Flexible — tower, workstation, or quiet rack-mount all acceptable
- **Power**: Standard Swiss residential (230V)
- **Aesthetics**: Not important

## Platform

- Evaluate both **NVIDIA/CUDA** and **Apple Silicon** (unified memory)
- NVIDIA/CUDA preferred for software ecosystem breadth, but Apple Silicon acceptable if NVIDIA breaks the budget
- No lock-in to specific inference software — the stack will evolve (Ollama, vLLM, llama.cpp, LM Studio, etc.)

## Preferred Retailers

### Primary (Swiss, local warranty/support)
- **digitec.ch** / **galaxus.ch** — Largest Swiss electronics retailer
- **brack.ch** — Major Swiss online retailer
- **Joule Performance** (jouleperformance.com) — Swiss builder, high-end workstations and multi-GPU systems
- **Apple Store CH** (apple.com/ch-de) — Direct, incl. refurbished section (up to 15% off, full warranty)
- **toppreise.ch** — Swiss price comparison across all retailers

### International (ships to Switzerland with warranty)
- **BIZON Tech** (bizon-tech.com) — Pre-built multi-GPU AI workstations, ships internationally
- **BRESSNER Technology** (shop.bressner.de) — German, GPU servers and HPC solutions
- **RECT** (rect.coreto-europe.com) — German, GPU server configurator
- **Supermicro eStore Europe** (store.supermicro.com) — Enterprise GPU servers

### Refurbished
- **Servershop24** (servershop24.de) — German, specialized refurbished server shop (lower risk than random marketplaces)

## Timeline

- Purchase: **February 2026** (this month)
- This is what's available on the market right now

## Access Pattern

- Remote access only (SSH / API / web UI from laptop)
- No monitor/keyboard needed on the inference machine
- Should function as a headless inference server

## Future Considerations

- This starts as a hobby/side-project setup but may grow into something more serious
- Possibility of sharing access with colleagues (not a hard requirement, but nice to have)
- MoE model trend is accelerating — expect more and better MoE coding models throughout 2026, further favoring high-bandwidth single-GPU setups over multi-GPU VRAM-maximizing builds
- Hardware upgrade path: start with a single high-VRAM GPU, add a second later if larger MoE models (200B+) require it
