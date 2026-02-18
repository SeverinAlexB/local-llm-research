# AI Inference Hardware Research

## Purpose

Living research repository for local AI inference hardware selection. Two audiences:
1. **Owner** — continuously updated to track hardware launches, pricing, and benchmarks for a purchase decision.
2. **Engineers** — shareable reference so others can evaluate hardware options and make their own decisions.

## Writing style

- **English only.**
- **TL;DR first, then deep dive.** Every file starts with a concise summary/recommendation, followed by dense technical detail (specs, formulas, tables).
- Maximize information density. Prefer tables over prose for comparisons.

## Repository structure

```
├── README.md                          # Navigation, reading order, scope
├── requirements.md                    # Performance targets, budget, constraints
├── decision-summary.md                # Ranked configurations with rationale
├── benchmarks.md                      # Cross-platform model/VRAM/inference reference
├── alternative-platforms-research.md  # Why 30+ platforms were ruled out
├── <platform>/                        # One folder per viable platform
│   ├── research.md                    #   Hardware deep-dive
│   ├── benchmarks.md                  #   Platform-specific performance data
│   └── build-guide.md                 #   Parts lists, pricing, assembly notes
```

New platforms follow this folder convention. File names within a platform folder are flexible — adapt to what makes sense for the platform.

## Consistency rules

After any content change, proactively update all affected cross-references:
- `decision-summary.md` — must reflect current recommendations and pricing.
- `README.md` — must list all platform folders and files.
- Platform files that reference each other — keep numbers and links in sync.

## Guardrails

- **Never auto-commit.** Only commit when explicitly asked.
- **No affiliate links** — never add tracking parameters or affiliate codes to URLs.
- **No speculative hardware** — do not include unannounced/unreleased products unless explicitly asked.