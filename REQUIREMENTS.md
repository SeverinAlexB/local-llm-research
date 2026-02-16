# AI Inference Hardware — Requirements

## Overview

Local AI inference hardware for home office use, primarily for AI-assisted coding with multi-agent workflows. Must support remote access (SSH/API from laptop — no local software needed).

## Use Cases (by priority)

1. **Code generation** — Primary workload. Large coding models (70B class) for main/complex tasks, smaller models (7B-32B) for sub-agents.
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

## Memory (VRAM)

- **Target**: 256 GB GPU-accessible RAM
- **Minimum**: 128 GB GPU-accessible RAM
- Must be GPU RAM (VRAM or unified memory) — not system RAM with CPU offloading
- Sufficient to hold a 70B model (main) + a 7B-32B model (sub-agents) + a vision model, all loaded concurrently

## Quantization

- FP8 preferred
- FP4 acceptable if needed for budget/memory reasons
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
- In ~1 year, 256 GB VRAM is expected to be sufficient for relevant workloads
