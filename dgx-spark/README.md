# DGX Spark / ASUS Ascent Path

> **TL;DR:** A 2-unit ASUS Ascent cluster (256 GB total, GB10 Blackwell) runs MoE coding models at **75 tok/s** for ~CHF 5,000. Turnkey, near-silent, no assembly. Dense 70B models are unusable (~5 tok/s). Buy the ASUS Ascent 1 TB variant — same chip as NVIDIA DGX Spark FE, CHF 1,026 cheaper per unit.

## Key specs — per unit (GB10 Superchip)

| Spec | Value |
|---|---|
| Memory | 128 GB unified LPDDR5X |
| Memory bandwidth | 273 GB/s |
| TDP | <240W (near-silent) |
| MoE performance | 50-55 tok/s (1 unit), **75 tok/s** (2-unit cluster) |
| Dense 70B performance | ~5 tok/s (unusable) |
| Unit price | [CHF 2,477](https://www.toppreise.ch/price-comparison/Complete-systems/ASUS-Ascent-GX10-GG0003BN-NVIDIA-Grace-Blackwell-90MS0371-M00030-p824149) (ASUS 1 TB) |
| 2-unit cluster | ~CHF 5,004 |
| 3-unit cluster | ~CHF 7,481 |

## Files

| File | Description |
|---|---|
| [research.md](research.md) | DGX Spark / ASUS Ascent deep-dive — specs, variants, clustering architecture, strengths & limitations |
| [benchmarks.md](benchmarks.md) | MoE and dense model performance, concurrent inference strategy |
| [build-guide.md](build-guide.md) | 2-unit and 3-unit cluster configurations with pricing and cable requirements |
