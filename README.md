# Research portfolio — Yuka Kust

Curated public summary of independent ML/research work done 2025–2026. Working code lives in private repos (`pt-moe-research`, `jippy`); this repo is the public-facing index for hiring contexts. Access to the private repos available on request.

## Highlights

### 🧪 [Era 10 — Matchbox LoRA: LoRA as persistent domain prior](era10_matchbox.md)

Empirical demonstration that LoRA acts as a persistent always-on domain prior, not a context-conditional switch. Same images, same model, same human voters: prompt-matching frame → 47% LoRA win, style frame → **80% LoRA win**. Bare prompt `factory worker` (no trigger word, no domain mention) renders in the LoRA's retro Soviet matchbox-label style; baseline produces a normal photo.

Methodological note: style-LoRA evaluations must use style-aware vote questions — content-matching framings systematically punish style adapters.

### 📐 [Phase 0 — Multi-LoRA composition: Arrow routing validation](phase0_results.md)

Arrow routing achieves **97% of oracle-routing accuracy in-domain** on a math5 benchmark (5 math LoRAs). Generation-jaccard reveals the true ranking that loss-on-validation initially obscured. Format-mimicry observation grounds the Layer-0 hypothesis. Foundation for hierarchical LoRA-guild routing in Phase 1.

### 🛠️ Jippy — personal AI Chrome extension (production)

Personal-AI Chrome extension shipped to friends + Chrome Web Store packaging. Stack: MV3 TypeScript extension + FastAPI Python backend + SQLite + filesystem blobs + JWT auth + token-bucket rate limiting (120 captions/min/user). Self-hosted infrastructure: Caddy edge on Hetzner + autossh reverse tunnel into Vadim Ubuntu app tier + Tailscale mesh + Vast.ai 3090 GPUs for VLM captioning. Pinterest crawler with sharded queries across 5 accounts + pHash dedup. Live at `gpu.social/jippy/`. (Source private — `github.com/yukakust/jippy`.)

### 🎨 FLUX jppy LoRA — production logo generator

15,815-image dataset, Unsloth + QLoRA training pipeline (2× faster, 70% less VRAM than plain QLoRA). 4.9MB adapter, loss 0.21 → 0.17 in ~7h. Trigger `jppy-logo`. Live at `gpu.social`.

## Background

Independent researcher and product builder, 2025–present. Prior: Co-founder/CEO Futudo (LLM B2C startup, MVP in 6 months, $45K angel raised, Product Hunt #3 of day). 12 years B2B sales, 14 years cross-functional team leadership at Yandex (S&P 500: GM Yandex Delivery Turkey → Head of Growth LatAm — 0→20K daily orders, #1 of 10+ international launches; Head of Commerce Yandex Smena; Sales Lead Yandex Pay) and Glovo (Regional Manager Belarus). Founder Gruzin.by (4th-largest fast-food chain in Belarus, $4M+ revenue). Founder Ezhik.by (food bank during Belarus protests, helped 4000+ families).

## Contact

- Email: kustyuka@gmail.com
- LinkedIn: https://www.linkedin.com/in/yuka-kust-b7964097/
- Live: https://gpu.social
