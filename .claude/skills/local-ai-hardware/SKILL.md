---
name: local-ai-hardware
description: "Guide developers through choosing local AI inference hardware — GPUs, Apple Silicon, complete builds. Covers VRAM sizing, memory bandwidth, KV cache/context window impact, dense vs MoE performance estimation, concurrent inference, and build config with live Swiss pricing via toppreise-cli. Use when the user asks about: which GPU for running AI locally, how fast a model will run, how much VRAM needed, dense vs MoE trade-offs, building an inference workstation, NVIDIA vs Apple Silicon, KV cache memory needs. Triggers: 'What GPU for Llama 70B?', 'How fast will this model run?', 'Mac Studio or NVIDIA?', 'VRAM for concurrent models?', 'Build an AI machine', 'RTX 5090 vs PRO 6000', 'VRAM for 128K context?'."
---

# Local AI Hardware Advisor

Guide developers through selecting hardware for local AI inference. Use real benchmark data, performance formulas, and live Swiss prices via `toppreise-cli`.

## Decision Workflow

1. **Understand the workload** — What models? Dense or MoE? What speed targets? How many concurrent streams? What context lengths?
2. **Estimate VRAM** — Model weights + KV cache (context-dependent) + concurrent models + overhead
3. **Estimate performance** — Use the bandwidth formulas to predict tok/s
4. **Select hardware** — Match VRAM, bandwidth, budget, noise, and power constraints
5. **Configure the build** — CPU, motherboard, PSU, case, cooling
6. **Fetch live prices** — Use `toppreise-cli` for current Swiss pricing

## The Bandwidth Formula — Most Important Concept

Token generation is **memory-bandwidth-bound**. Every token requires reading model weights from VRAM:

```
Dense model:  tok/s ≈ Memory Bandwidth (GB/s) / Model Size in VRAM (GB)
MoE model:    tok/s ≈ Memory Bandwidth (GB/s) / Active Parameters Size in VRAM (GB)
```

Real-world efficiency is ~60-75% of theoretical due to overhead (KV cache access, attention computation, synchronization).

**Example — Dense 70B Q4 (~36 GB) on RTX PRO 6000 (1,792 GB/s):**
Theoretical: 1,792 / 36 ≈ 50 tok/s. Real: ~32 tok/s (64% efficiency).

**Example — MoE GPT-OSS-120B (~20B active, ~10 GB active weights Q4) on RTX PRO 6000:**
Theoretical: 1,792 / 10 ≈ 179 tok/s. Real: ~134 tok/s (75% efficiency).

## Dense vs MoE — How Architecture Changes Hardware Needs

**Dense models** (Llama 70B, Qwen2.5-Coder 72B): Every parameter is read per token. Speed limited by total model size / bandwidth.

**MoE models** (GPT-OSS-120B, Qwen3-30B-A3B, MiniMax-M2.5): Only active expert parameters are read per token. Speed depends on active params, not total. But **all parameters** must fit in VRAM.

Key implication: MoE models need large VRAM (to store all params) but run fast (only reading active params). A 120B MoE model needs ~65 GB VRAM but runs at speeds comparable to a ~20B dense model. Since early 2025, most frontier coding models use MoE architecture.

## Context Window Requirements for Coding

Coding agents (e.g., Claude Code, Cursor, aider) typically require **200K+ token context windows** to hold project files, conversation history, tool outputs, and system prompts simultaneously. This is a hard constraint that affects both **model selection** and **VRAM sizing**.

### Model Context Window Support

Not all models support 200K context. Always verify before recommending:

| Model | Max Context | Type | Notes |
|---|---|---|---|
| Qwen3-Coder-480B-A35B | **256K** (1M with YaRN) | MoE (35B active) | Best coding model with long context. ~240 GB Q4 weights — needs multi-GPU or Apple 512 GB. |
| Qwen3-Coder-30B-A3B | **256K** (1M with YaRN) | MoE (3.3B active) | Fits on single GPU. Good sub-agent. May lack quality for main agent at complex tasks. |
| GLM-4.7 / GLM-5 | **200K** | Dense/MoE | Meets 200K threshold. Check local availability and quantized formats. |
| GPT-OSS-120B | **128K** | MoE (20B active) | Does NOT meet 200K. Fast (134 tok/s) but context-limited for coding agents. |
| Llama 3.1/3.3 70B | **128K** | Dense | Does NOT meet 200K. |
| Qwen2.5-Coder 72B | **128K** (1M with YaRN) | Dense | 128K native; extended context via YaRN may degrade quality. |
| Qwen3-30B-A3B (base) | **256K** | MoE (3.3B active) | General-purpose, not code-specialized. |

**Key finding**: For a 200K+ context coding agent, the primary local options are:
1. **Qwen3-Coder-480B-A35B** — Best quality, but ~240 GB Q4 weights requires 256+ GB VRAM (Apple Silicon 512 GB, or 3x RTX PRO 6000)
2. **Qwen3-Coder-30B-A3B** — Fits on a single 96 GB card with room for 200K KV cache. Smaller model — good for many tasks, may not match frontier quality on complex reasoning.
3. **GLM-5** — 200K context, strong coding benchmarks. Verify local deployment support.
4. **Qwen2.5-Coder 72B with YaRN** — Extended context may work but with quality degradation beyond 128K.

Models like GPT-OSS-120B (128K max) are excellent for speed but fall short on context length for full coding agent workflows.

## VRAM Sizing

Total VRAM needed = model weights + KV cache + framework overhead, summed across all concurrently loaded models.

### Model Weights

```
Q4/GGUF:  params_billions × 0.5 GB   (70B → ~35 GB)
FP8:      params_billions × 1.0 GB   (70B → ~70 GB)
FP16:     params_billions × 2.0 GB   (70B → ~140 GB)
```

For MoE models, use total params (not active) since all weights must reside in VRAM.

### KV Cache — The Hidden VRAM Cost

The KV (Key-Value) cache stores attention state for each token in the context. It grows **linearly with context length** and can dominate VRAM usage at long contexts.

**Formula:**

```
KV cache per request (bytes) = 2 × num_layers × num_kv_heads × head_dim × context_length × bytes_per_element

Total KV cache = KV_per_request × num_concurrent_requests
```

- `2` = one K tensor + one V tensor per layer
- `num_kv_heads` = number of key/value heads (often fewer than query heads due to GQA)
- `head_dim` = typically 128 for most modern models
- `bytes_per_element` = 2 (FP16), 1 (FP8/INT8), or 0.5 (INT4)

**Practical KV cache sizes for a 70B-class model (80 layers, 8 KV heads, 128 dim, FP16):**

| Context Length | KV Cache per Request (FP16) | KV Cache per Request (INT8) | 3 Concurrent (FP16) |
|---|---|---|---|
| 8K | ~2.5 GB | ~1.3 GB | ~7.5 GB |
| 32K | ~10 GB | ~5 GB | ~30 GB |
| 128K | ~40 GB | ~20 GB | ~120 GB |
| **200K** | **~65 GB** | **~32 GB** | **~195 GB** |

**Smaller MoE models have smaller KV caches.** Qwen3-Coder-30B-A3B (48 layers, 4 KV heads, 128 dim):

| Context Length | KV Cache per Request (FP16) | KV Cache per Request (INT8) |
|---|---|---|
| 8K | ~0.8 GB | ~0.4 GB |
| 32K | ~3.1 GB | ~1.6 GB |
| 128K | ~12.6 GB | ~6.3 GB |
| **200K** | **~19.7 GB** | **~9.8 GB** |

**Why this matters — especially at 200K context (coding agent minimum):**
- A 70B Q4 model (~35 GB weights) + single 200K KV cache in FP16 (~65 GB) = **~100 GB — does NOT fit on a 96 GB card.** INT8 KV cache (~32 GB) brings it to ~67 GB — fits.
- Qwen3-Coder-30B-A3B Q4 (~18 GB weights) + 200K KV cache FP16 (~20 GB) = **~38 GB — fits easily on 96 GB** with room for sub-agents.
- At 200K context, **KV cache quantization (INT8/FP8) is essentially mandatory** for 70B+ models on cards with less than 128 GB VRAM.
- Apple Silicon's 256-512 GB unified memory becomes significantly more attractive for long-context workloads — no KV cache pressure.

**Reducing KV cache size:**
- **KV cache quantization (INT8/FP8)**: Halves memory vs FP16 with minimal quality loss. **Essential for 200K context on NVIDIA GPUs.** Supported by vLLM, SGLang, and llama.cpp.
- **GQA (Grouped Query Attention)**: Modern models use far fewer KV heads than query heads (e.g., 8 KV vs 64 query), already reducing KV cache 8x vs full MHA. Already built into Llama 3, Qwen3, etc.
- **PagedAttention** (vLLM): Allocates KV cache on-demand rather than pre-allocating for max context, reducing waste.
- **Choose models with fewer KV heads**: Qwen3-Coder-30B-A3B (4 KV heads) has ~3x smaller KV cache per token than a 70B model (8 KV heads, more layers).

### VRAM Budget Examples — 200K Context (Coding Agent)

```
Scenario A: RTX PRO 6000 (96 GB), Qwen3-Coder-30B-A3B as main agent, 200K context
├── Qwen3-Coder-30B-A3B Q4 (main agent):  ~18 GB weights
├── KV cache (1 × 200K, INT8):            ~10 GB
├── Qwen3-Coder-30B-A3B Q4 (sub-agent):   ~18 GB weights  (can share weights with main)
├── KV cache (2 sub × 32K, INT8):         ~3 GB
├── Qwen2.5-VL-7B Q4 (vision):             ~5 GB
├── Framework overhead:                     ~2 GB
└── Total: ~56 GB — fits comfortably on 96 GB. Room for more streams.

Scenario B: RTX PRO 6000 (96 GB), 70B dense model, 200K context
├── Qwen2.5-Coder 72B Q4 (main agent):    ~36 GB weights
├── KV cache (1 × 200K, FP16):            ~65 GB  ← DOES NOT FIT!
├── KV cache (1 × 200K, INT8):            ~32 GB  ← Total: 68 GB — fits but no room for sub-agents
└── Verdict: 70B + 200K context requires INT8 KV cache and leaves almost no headroom.

Scenario C: Mac Studio M3 Ultra (256 GB), Qwen3-Coder-480B-A35B, 200K context
├── Qwen3-Coder-480B-A35B Q4 (main):      ~240 GB weights  ← DOES NOT FIT in 256 GB with KV cache
└── Verdict: Needs 512 GB config (~CHF 8,400) or a smaller model.

Scenario D: Mac Studio M3 Ultra 256 GB, Qwen3-Coder-30B-A3B, 200K context
├── Qwen3-Coder-30B-A3B Q4 (main):        ~18 GB weights
├── KV cache (1 × 200K, FP16):            ~20 GB
├── Sub-agents + vision + more streams:    ~40 GB
├── OS + system:                           ~10 GB
└── Total: ~88 GB of 256 GB — massive headroom for concurrent long-context streams
```

## Quantization Quick Reference

| Format | Bits/param | VRAM per 70B | Quality | Notes |
|---|---|---|---|---|
| FP16 | 16 | ~140 GB | Best | Fine-tuning/research only |
| FP8 | 8 | ~70 GB | Near-lossless | Good default when VRAM allows |
| Q4/GGUF | 4 | ~35 GB | Good | Default for local inference |
| NVFP4 | 4 | ~35 GB | Good | Native acceleration on Blackwell GPUs |

## Concurrent Inference

Multiple streams sharing a GPU compete for memory bandwidth **and** consume additional KV cache VRAM per stream:

- **Same GPU**: Total throughput scales ~linearly up to saturation; per-stream tok/s drops proportionally
- **Separate GPUs**: Zero bandwidth contention — each model gets full card bandwidth
- **Rule of thumb**: At 8 concurrent streams, expect ~25-35% of single-stream per-request speed
- **KV cache stacks**: Each concurrent request allocates its own KV cache. At 200K context, even a single Qwen3-30B stream uses ~10-20 GB for KV cache. Multiple long-context streams can exhaust VRAM fast.

Inference engine recommendations:
- **vLLM / SGLang**: Best for NVIDIA multi-stream serving. Continuous batching. SGLang more stable under load.
- **vllm-mlx**: Best for Apple Silicon concurrent serving (21-87% over llama.cpp)
- **Ollama**: Easy setup, poor scaling beyond 3-5 streams

## Multi-GPU Considerations

| Interconnect | Bandwidth | Scaling |
|---|---|---|
| PCIe 4.0 x16 | ~32 GB/s | ~1.5-1.7x for 2 GPUs |
| PCIe 5.0 x16 | ~64 GB/s | ~1.6-1.8x for 2 GPUs |
| NVLink (PRO 6000/A6000/3090) | ~112.5 GB/s | ~1.7-1.9x for 2 GPUs |

- **Tensor Parallelism**: Splits layers across GPUs. Lower latency. Needs fast interconnect.
- **Independent models on separate GPUs**: No interconnect needed — just separate inference servers.

Platform requirements:
- **2 GPUs**: AM5 X870E (PCIe x8/x8) or TRX50 Threadripper (x16/x16, NVLink)
- **3-4 GPUs**: Threadripper PRO WRX90 (128 PCIe 5.0 lanes)

## GPU Quick Reference

| GPU | VRAM | Bandwidth | TDP | NVLink | ~CHF |
|---|---|---|---|---|---|
| **RTX PRO 6000 Blackwell** | 96 GB GDDR7 | 1,792 GB/s | 300W | Yes | 6,400-7,700 |
| **RTX 5090** | 32 GB GDDR7 | 1,792 GB/s | 575W | No | 2,100-3,500 |
| **RTX 4090** | 24 GB GDDR6X | 1,008 GB/s | 450W | No | ~2,800 (scarce) |
| **RTX 3090** (used) | 24 GB GDDR6X | 936 GB/s | 350W | 2-way | ~550-650 |
| **RTX A6000** | 48 GB GDDR6 | 768 GB/s | 300W | 2-way | 1,900-3,300 (used) |
| **A100 80GB** (used) | 80 GB HBM2e | 2,039 GB/s | 300W | Yes | 5,100-7,000 |
| **Mac M3 Ultra** | 256 GB unified | 819 GB/s | ~250W | N/A | ~6,100 (256GB) |
| **Mac M4 Max** | 128 GB unified | 546 GB/s | ~150W | N/A | ~3,200 (128GB) |

## Key Benchmarks

### MoE Models (the fast path)

| Hardware | Model (MoE) | Active Params | tok/s |
|---|---|---|---|
| RTX PRO 6000 | GPT-OSS-120B Q4 | ~20B | **134** |
| RTX PRO 6000 | GPT-OSS-20B Q4 | ~5B | **185** |
| RTX PRO 6000 | Qwen3-30B-A3B Q4 | 3.3B | **~150-200** |
| Mac M3 Ultra | GPT-OSS-120B Q4 | ~20B | **69** (1 stream) / 19 (8 streams) |
| Mac M3 Ultra | Qwen3-30B-A3B Q4 | 3.3B | **84** (1 stream) / 25 (8 streams) |

### Dense Models (slower, still relevant as fallbacks)

| Hardware | Model (Dense) | tok/s (Q4) |
|---|---|---|
| RTX PRO 6000 | 70B class | **29-32** |
| RTX PRO 6000 | 32B class | **56-64** |
| 2x RTX 5090 | 70B class | **~27** |
| A100 80GB | 70B class | **~35-40** |
| Mac M3 Ultra | 70B class (MLX) | **~15-25** |
| Mac M3 Ultra | 32B class | **~33-41** |

## Apple Silicon vs NVIDIA — Key Trade-offs

| Factor | Apple Silicon (M3 Ultra) | NVIDIA (RTX PRO 6000) |
|---|---|---|
| MoE speed (single stream) | ~69 tok/s | ~134 tok/s (2x faster) |
| Dense 70B speed | ~15-25 tok/s | ~29-32 tok/s |
| Concurrent scaling (8 streams) | ~70% degradation | ~25-40% degradation |
| Max memory | 512 GB (soldered) | 96 GB per card (expandable) |
| **200K context headroom** | **Excellent** (256 GB handles KV cache in FP16) | **Tight** (96 GB requires INT8 KV cache) |
| Noise | Silent | Moderate (blower fan) |
| Power | ~200-300W | ~400W |
| Setup | Zero (turnkey) | Build required |
| Upgrade path | None | Add 2nd GPU via NVLink |
| Software | MLX/Metal only | CUDA (broadest ecosystem) |

## Inference Software

| Engine | Multi-GPU TP | Continuous Batching | Apple Silicon | Best For |
|---|---|---|---|---|
| **vLLM** | Yes | Yes | vllm-mlx | Production multi-GPU serving |
| **SGLang** | Yes | Yes | No | Same as vLLM, more stable under load |
| **Ollama** | No | Limited | Yes (Metal) | Quick setup, development |
| **llama.cpp** | No (layer split) | Limited | Yes (Metal) | Single-user low latency |
| **MLX** | No | Yes (vllm-mlx) | Native | Apple Silicon native, 20-30% faster than llama.cpp |

## Build Essentials

When recommending a full build:

- **PSU**: Size for 1.5x total system power. 850-1000W for single GPU, 1600W for dual GPU.
- **CPU**: Ryzen 9 9950X for 1-2 GPUs (AM5). Threadripper 7960X for 2+ GPUs or NVLink (TRX50).
- **Case**: Sound-dampened (Fractal Design Define 7 / 7 XL). Full tower for multi-GPU.
- **Cooling**: Air cooling sufficient — CPU is not the inference bottleneck. Noctua NH-D15 or equivalent.
- **Storage**: 2 TB NVMe minimum. Model downloads are large.
- **OS**: Ubuntu Server headless, accessed via SSH.
- **Noise**: RTX PRO 6000 WS at 300W is manageable. Passive-cooled datacenter cards (L40S, A100) are NOT suitable for home office without custom cooling.

## Fetching Live Prices

Always use `toppreise-cli` for current Swiss pricing:

```bash
toppreise-cli search "RTX PRO 6000" --sort price-asc --limit 5
toppreise-cli product <id-or-url> --section offers
toppreise-cli search "AMD Ryzen 9 9950X" --sort price-asc
```

Include the toppreise.ch link and current CHF price for every hardware component mentioned.

## Example Interactions

**"I need a local coding agent with 200K context. What hardware?"**
→ First check model support: Qwen3-Coder-30B-A3B (256K) or GLM-5 (200K). Size VRAM: model weights + KV cache at 200K (use INT8 on NVIDIA). Qwen3-Coder-30B-A3B Q4 (~18 GB) + 200K KV INT8 (~10 GB) = ~28 GB — fits easily on RTX PRO 6000 (96 GB) or Mac M3 Ultra (256 GB). Note: GPT-OSS-120B only supports 128K — insufficient.

**"How fast will GPT-OSS-120B run on a Mac Studio M3 Ultra?"**
→ MoE: 120B total, ~20B active. Measured: ~69 tok/s single-stream, ~19 tok/s at 8 concurrent. Fast, but only supports 128K context — insufficient for 200K coding agent workflows.

**"I need 3 concurrent coding agents with 200K context"**
→ 3 × Qwen3-Coder-30B-A3B: ~18 GB weights (shared) + 3 × 200K KV cache (~30 GB INT8 / ~60 GB FP16). Total: ~48-78 GB. Fits on 96 GB PRO 6000 with INT8 KV cache. Apple Silicon 256 GB handles this with FP16 KV cache and room to spare.

**"Mac Studio or NVIDIA workstation?"**
→ NVIDIA: 2x faster MoE inference, better concurrent scaling, CUDA ecosystem, upgradeable. Apple: silent, massive unified memory (256-512 GB ideal for long context), no build effort. For 200K+ context with multiple streams, Apple's memory advantage narrows the gap despite slower bandwidth.

**"What GPU do I need for Llama 3.3 70B?"**
→ Dense 70B, 128K max context (not 200K). Q4 ≈ 34 GB weights. At 128K: ~40 GB KV cache FP16 / ~20 GB INT8 per stream. Need 96+ GB card. Estimate ~29-32 tok/s on RTX PRO 6000.
