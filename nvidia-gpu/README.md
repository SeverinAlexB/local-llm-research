# NVIDIA GPU Path

> **TL;DR:** The RTX PRO 6000 Blackwell (96 GB GDDR7, 1,792 GB/s) is the top recommendation. A single card runs MoE coding models at **134 tok/s** — far exceeding the 60-80 tok/s target. Full build ~CHF 9,500. Dense 70B models cap at ~32 tok/s; MoE is the path to speed.

## Key specs — RTX PRO 6000

| Spec | Value |
|---|---|
| VRAM | 96 GB GDDR7 ECC |
| Memory bandwidth | 1,792 GB/s |
| TDP | 300W |
| NVLink | Yes (expandable to 2 GPUs / 192 GB) |
| MoE performance | 134 tok/s (GPT-OSS-120B Q4) |
| Dense 70B performance | ~32 tok/s (Q4) |
| GPU price | [CHF 7,691](https://www.toppreise.ch/price-comparison/Graphics-cards/NVIDIA-RTX-Pro-6000-Blackwell-Workstation-900-5G144-2200-000-p833823) |
| Full build price | ~CHF 9,500 |

## Files

| File | Description |
|---|---|
| [research.md](research.md) | Deep-dive on every NVIDIA GPU considered (RTX PRO 6000, RTX 5090, RTX 4090, A6000, A100, H100) |
| [benchmarks.md](benchmarks.md) | Token generation speeds for 7B–70B dense and MoE models, concurrent inference data |
| [build-guide.md](build-guide.md) | Parts lists, Swiss prices, and shop links for recommended configurations |
| [vendor-options.md](vendor-options.md) | Pre-built workstation vendors (BIZON, BRESSNER, RECT) for buyers who prefer not to build |
