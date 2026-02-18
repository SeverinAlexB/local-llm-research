# Coding Model Quality Benchmarks

> Compiled February 18, 2026. Benchmarks measure **model quality** — how well models solve real coding tasks. For speed (tok/s) and VRAM requirements, see [benchmarks.md](benchmarks.md).

## TL;DR

**96 GB (single RTX PRO 6000) runs good but not frontier-quality coding models.** The best local model that fits — Qwen3-Coder-Next-80B at 70.6% SWE-Bench Verified — is 10 points behind the best open-weight model (MiniMax-M2.5 at 80.2%) and 10 points behind the proprietary leader (Claude Opus 4.5 at 80.9%). On LiveCodeBench (competitive programming), the gap narrows: GPT-OSS-120B at 83.2% is within 4 points of frontier. The quality gap is largest on real-world software engineering benchmarks (SWE-Bench, Aider) and smallest on algorithmic coding.

**Best coding model by VRAM tier:**

| VRAM Available | Best Model | SWE-Bench Verified | LiveCodeBench v6 |
|---|---|---|---|
| **96 GB** (1× PRO 6000) | Qwen3-Coder-Next-80B-A3B (~45 GB Q4) | 70.6% | — |
| **96 GB** (1× PRO 6000) | GPT-OSS-120B (~65 GB Q4) | 62.4% | 83.2% |
| **128 GB** (PRO 6000 + 5090) | MiniMax-M2.5 (~130 GB NVFP4) | **80.2%** | 65% |
| **192 GB** (2× PRO 6000) | MiniMax-M2.5 (~130 GB) + sub-agents | **80.2%** | 65% |
| **256 GB+** (Mac Studio / DGX Spark cluster) | Qwen3.5-397B-A17B (~212 GB Q4) | 76.4% | 83.6% |

---

## 1. SWE-Bench Verified (Real-World Software Engineering)

500 human-validated GitHub issues. Models must understand codebases, locate bugs, and produce working patches. The most relevant benchmark for multi-agent coding workflows.

| Model | Type | VRAM (Q4) | SWE-Bench Verified | Fits 96 GB? |
|---|---|---|---|---|
| Claude Opus 4.5 | Proprietary | — | **80.9%** | — |
| MiniMax-M2.5 | MoE 230B / 10B active | ~130 GB | **80.2%** | No |
| Claude Opus 4.6 | Proprietary | — | **80.8%** | — |
| GPT-5.2 | Proprietary | — | **80.0%** | — |
| Claude Sonnet 4.5 | Proprietary | — | **77.2%** | — |
| Qwen3.5-397B-A17B | MoE 397B / 17B active | ~212 GB | **76.4%** | No |
| Gemini 3 Pro | Proprietary | — | **76.2%** | — |
| GPT-5 | Proprietary | — | **74.9%** | — |
| DeepSeek-V3.2 | MoE 671B / 37B active | ~386 GB | **73.0%** | No |
| **Qwen3-Coder-Next-80B** | **MoE 80B / 3B active** | **~45 GB** | **70.6%** | **Yes** |
| GPT-OSS-120B | MoE 120B / 20B active | ~65 GB | **62.4%** | **Yes** |
| DeepSeek-R1-0528 | MoE 671B / 37B active | ~386 GB | ~57.6% | No |
| Qwen3-Coder-30B-A3B | MoE 30B / 3.3B active | ~18 GB | ~51% | **Yes** |
| GPT-4o | Proprietary | — | 33.2% | — |

**Key insight:** The best model that fits on 96 GB (Qwen3-Coder-Next at 70.6%) is **10.3 points behind** the best open-weight model (MiniMax-M2.5 at 80.2%). Moving from 96 GB to 128 GB unlocks MiniMax-M2.5 — the single biggest quality jump available.

Sources: [SWE-Bench Leaderboard](https://www.swebench.com/), [llm-stats.com](https://llm-stats.com/benchmarks/swe-bench-verified), [marc0.dev](https://www.marc0.dev/en/leaderboard), [Clarifai GPT-OSS analysis](https://www.clarifai.com/blog/openai-gpt-oss-benchmarks-how-it-compares-to-glm-4.5-qwen3-deepseek-and-kimi-k2)

---

## 2. LiveCodeBench v6 (Competitive Programming)

1,000+ fresh problems from LeetCode, AtCoder, CodeForces. Contamination-free — problems post-date model training. Tests algorithmic reasoning, not real-world software engineering.

| Model | LiveCodeBench v6 | Fits 96 GB? |
|---|---|---|
| GPT-5 / GPT-5.2 | **~87-89%** | — |
| Claude Opus 4.5 | **~87%** | — |
| Qwen3.5-397B-A17B | **83.6%** | No |
| GPT-OSS-120B | **83.2%** | **Yes** |
| DeepSeek-V3 | **73.3%** | No |
| Gemini 2.5 Pro | 70.4% (v5) | — |
| MiniMax-M2.5 | **65%** | No |

**Key insight:** On competitive programming, GPT-OSS-120B (83.2%) is genuinely close to frontier and even beats larger open-weight models. The 96 GB ceiling barely hurts here.

Sources: [LiveCodeBench Leaderboard](https://livecodebench.github.io/leaderboard.html), [Artificial Analysis](https://artificialanalysis.ai/models/gpt-oss-120b)

---

## 3. Aider Polyglot (Practical Code Editing)

225 Exercism exercises across C++, Go, Java, JavaScript, Python, Rust. Tests ability to edit existing code in real files — closest to daily coding assistant use.

| Model | Aider Polyglot | Fits 96 GB? |
|---|---|---|
| Claude Opus 4.5 | **89.4%** | — |
| GPT-5 | **88.0%** | — |
| Gemini 2.5 Pro | **83.1%** | — |
| Claude Sonnet 4.5 | **78.8%** | — |
| DeepSeek-R1 | **71.4%** | No |
| GPT-OSS-120B | **68.4%** (diff) | **Yes** |
| DeepSeek-V3 | **55%** (whole) | No |
| GPT-OSS-120B | **41.8%** (whole) | **Yes** |

**Key insight:** The gap is largest here. GPT-OSS-120B (68.4% diff) is **21 points behind** Claude Opus 4.5 (89.4%). For daily coding assistance, the quality difference between local and proprietary is most noticeable.

Sources: [Aider Polyglot Leaderboard](https://aider.chat/docs/leaderboards/)

---

## 4. Practical Coding Evaluation (16x Engineer)

Real-world tasks: markdown, TypeScript, debugging, visualization. Small sample but directly measures practical coding quality.

| Model | Score (/10) | Type |
|---|---|---|
| Claude Opus 4 | **9.2** | Proprietary |
| Claude Sonnet 4 | **8.75** | Proprietary |
| Grok 4 | **8.4** | Proprietary |
| GPT-OSS-120B | **8.3** | Open-weight (**fits 96 GB**) |
| GPT-4.1 | **8.2** | Proprietary |
| Gemini 2.5 Pro | **7.25** | Proprietary |
| Qwen3-Coder | **6.8** | Open-weight |
| DeepSeek-V3 | **6.7** | Open-weight |

**Key insight:** GPT-OSS-120B (8.3) is competitive with proprietary models on practical tasks — within 1 point of Claude Opus 4 (9.2). This benchmark is more favorable to local models than SWE-Bench.

Source: [16x Engineer Evaluation](https://eval.16x.engineer/blog/gpt-oss-120b-coding-evaluation-results)

---

## 5. Summary: Where the 96 GB Ceiling Hurts

| Benchmark Type | 96 GB Gap vs Frontier | Verdict |
|---|---|---|
| **SWE-Bench** (real-world SWE) | −10 to −18 points | **Significant gap** |
| **Aider** (code editing) | −21 points | **Largest gap** |
| **LiveCodeBench** (algorithms) | −4 points | **Competitive** |
| **Practical eval** (mixed tasks) | −0.9 points | **Near parity** |

The gap is task-dependent:
- **Algorithmic/competitive coding:** 96 GB models are close to frontier. GPT-OSS-120B excels here.
- **Real-world software engineering:** Significant gap. SWE-Bench and Aider reward the ability to navigate large codebases and produce precise patches — areas where larger models (MiniMax-M2.5 at 130 GB, Qwen3.5-397B at 212 GB) clearly outperform.
- **Practical mixed tasks:** GPT-OSS-120B holds up well, scoring 8.3/10 vs 9.2/10 for Claude Opus 4.

### Implications for Hardware Decisions

- **96 GB (1× PRO 6000):** Good for competitive programming and general coding. Not frontier for agentic SWE tasks. Best model: Qwen3-Coder-Next-80B (70.6% SWE-Bench) or GPT-OSS-120B (83.2% LiveCodeBench).
- **128 GB (PRO 6000 + 5090):** Unlocks MiniMax-M2.5 (80.2% SWE-Bench) — the single biggest quality jump. Near-frontier agentic coding.
- **192 GB (2× PRO 6000):** Runs MiniMax-M2.5 comfortably with room for sub-agents. No additional model quality unlocked vs 128 GB, but better concurrent performance.
- **256 GB+ (Mac Studio / DGX Spark cluster):** Unlocks Qwen3.5-397B-A17B (76.4% SWE-Bench, 83.6% LiveCodeBench). But slower inference (69 tok/s Mac vs 134 tok/s PRO 6000 on MoE).

---

## Caveats

1. **Benchmark versions matter.** LiveCodeBench v5 and v6 produce different scores; don't compare across versions.
2. **Scaffold/agent matters for SWE-Bench.** Scores depend heavily on the agentic scaffold (OpenHands, Agentless, etc.), not just the model. The same model can score ±10% depending on the framework.
3. **Quantization degrades quality.** All "fits on 96 GB" estimates assume Q4. Running at Q4 vs FP16 may degrade benchmark scores by 1-5%, though this is rarely measured.
4. **HumanEval/MBPP are saturated.** Most frontier models score 85-95% on these classic benchmarks, so they no longer differentiate. LiveCodeBench and SWE-Bench are far more informative.
5. **MiniMax-M2.5 uses NVFP4** (~130 GB), not Q4_K_M. Actual VRAM depends on quantization format and inference engine.
6. **Model quality evolves fast.** New MoE models that fit on 96 GB and close the SWE-Bench gap could appear at any time.

---

## Sources

### Benchmark Leaderboards
- [SWE-Bench Official Leaderboard](https://www.swebench.com/)
- [SWE-Bench Verified (llm-stats.com)](https://llm-stats.com/benchmarks/swe-bench-verified)
- [SWE-Bench Breakdown (marc0.dev)](https://www.marc0.dev/en/leaderboard)
- [LiveCodeBench Leaderboard](https://livecodebench.github.io/leaderboard.html)
- [Aider Polyglot Leaderboard](https://aider.chat/docs/leaderboards/)
- [16x Engineer Coding Evaluation](https://eval.16x.engineer/blog/gpt-oss-120b-coding-evaluation-results)

### Model-Specific
- [GPT-OSS Benchmarks (Clarifai)](https://www.clarifai.com/blog/openai-gpt-oss-benchmarks-how-it-compares-to-glm-4.5-qwen3-deepseek-and-kimi-k2)
- [GPT-OSS-120B LiveCodeBench (Artificial Analysis)](https://artificialanalysis.ai/models/gpt-oss-120b)
- [MiniMax-M2.5 Announcement](https://www.minimax.io/news/minimax-m25)
- [Qwen3.5-397B-A17B Model Card](https://huggingface.co/Qwen/Qwen3.5-397B-A17B)
- [Qwen3-Coder-Next Announcement](https://qwen.ai/blog?id=qwen3-coder-next)
- [Claude Opus 4.5 SWE-Bench](https://www.theunwindai.com/p/claude-opus-4-5-scores-80-9-on-swe-bench)
- [Claude 4 Benchmarks](https://huggingface.co/blog/Laser585/claude-4-benchmarks)
- [Best AI for Coding 2026 (marc0.dev)](https://www.marc0.dev/en/blog/best-ai-for-coding-2026-swe-bench-breakdown-opus-4-6-qwen3-coder-next-gpt-5-3-and-what-actually-matters-1770387434111)
