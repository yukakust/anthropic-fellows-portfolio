# Research portfolio — Yuka Kust

Curated public summary of independent ML/research work done 2025–2026. Working code lives in private repos (`pt-moe-research`, `jippy`); this repo is the public-facing index. Access to private repos available on request.

## Safety thesis (read first)

### 🛡️ [Personal models as anti-concentration safety infrastructure](safety_thesis.md)

The dominant AI safety risk over the next decade is **concentration**: one or two foundation models serving a billion people, with values and failure modes centralized in a small number of corporate weights. Behavioural alignment work addresses *how* a centralized model behaves; it does not address *that* the model is centralized — that is a structural problem requiring structural mitigations.

The architectural bet I am building toward: **distributed AI ownership** via personal models — small adapters trained on an individual's data, sitting on top of (but not subordinate to) a shared backbone. Each user holds their own slice of cognition. Concentration containment, interpretability tractability, capability bounding, and sovereignty fall out of the architecture. The four research findings below are concrete steps toward that thesis.

## Highlights

### 🧪 [Era 10 — LoRA as persistent domain prior](era10_matchbox.md)

Empirical demonstration that LoRA acts as a persistent always-on domain prior, not a context-conditional switch. Same images, same model, same human voters: prompt-matching frame → 47% LoRA win, style frame → **80% LoRA win**. Bare prompt `factory worker` (no trigger word, no domain mention) renders in the LoRA's retro Soviet matchbox-label style; baseline produces a normal photo. Validates personal-LoRAs as *strong* persistent interventions — the right unit for studying personalization mechanistically.

Methodological note: style-LoRA evaluations must use style-aware vote questions — content-matching framings systematically punish style adapters.

### 🗣️ [Phase 0 — Personal voice recovery: 4-way SFT eval](phase0_yuka_voice_eval.md)

A 68 MB LoRA recovers individual communication style at parity with full fine-tuning (1.7 GB) on held-out personal Telegram + Claude Code session data. **25× compression, no quality loss.** Phone-class hardware deployable. Moves personal-AI from "research curiosity" to "feasible deployment target today" — a 68 MB per-user delta is shippable, a 1.7 GB delta is not.

### 🥟 [Georgian-cuisine eval — 125M specialist ties (and routed pair beats) frontier Claude](georgian_cuisine_eval.md)

A 125 M model SFT-trained on **99 texts** (~42K words) about Georgian cuisine ties frontier Claude on direct domain retrieval (Claude 6 / Specialist 7 / Tie 7 across 20 blind questions). A two-specialist + router variant (Georgian + Atatürk) **beats the same Claude 5–3 on direct test** with 8 ties. Empirical evidence that on a narrow domain the user actually cares about, a tiny personal-scale model is not worse than the centralized frontier model — the "concentration containment" pillar of the safety thesis is operational, not aspirational.

### 📐 [Phase 0 — Multi-LoRA composition: Arrow routing validation](phase0_results.md)

Arrow routing achieves **97% of oracle-routing accuracy in-domain** on a math5 benchmark (5 math LoRAs). Generation-jaccard reveals the true ranking that loss-on-validation initially obscured. Format-mimicry observation grounds the Layer-0 hypothesis. Required infrastructure for serving many personal/specialist adapters per user without ground-truth routing oracles.

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
