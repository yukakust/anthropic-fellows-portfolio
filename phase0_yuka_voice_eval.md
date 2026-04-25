# Phase 0 — 4-way SFT eval: personal-voice recovery

Empirical validation that a small LoRA (68 MB) recovers an individual's communication style at parity with full fine-tuning (1.7 GB). 25× compression, no quality loss on the eval. Phone-class hardware deployable.

## TL;DR

A 4-way blind eval comparing four model variants on the same set of held-out personal conversation pairs (telegram + Claude Code engineering sessions):

| Variant | Size | Parameters trained |
|---|---|---|
| **Base** Qwen3.5-0.8B | — | 0 (no fine-tune) |
| **A1 — LoRA reply mode** | 68 MB | rank-r adapter |
| **A2 — LoRA ask mode** | 68 MB | rank-r adapter |
| **B1 — Full fine-tune reply** | 1.7 GB | all params |
| **B2 — Full fine-tune ask** | 1.7 GB | all params |

**Headline result:** LoRA outputs are scored at parity with full fine-tune outputs in blind evaluation, while base Qwen3.5-0.8B produces visibly off-character "helpful AI" responses on the same prompts. The ~25× compression (68 MB vs 1.7 GB) does not cost evaluation quality.

For a personal-AI design point — one model per user, deployable on phone-class hardware — this is the difference between *feasible* and *not feasible*. A 1.7 GB per-user weight delta is operationally hard to ship; a 68 MB per-user weight delta is trivially shippable.

## Setup

| Component | Detail |
|---|---|
| Base model | Qwen3.5-0.8B-Instruct |
| Training data | Personal Telegram corpus + Claude Code session corpus (mixed reply / ask modes) |
| LoRA config | rank=8, masked-prompt loss (loss only on assistant tokens), AdamW |
| Full-FT config | All parameters, same data, same masked-prompt loss |
| Eval pairs | 100 held-out (context, ground-truth reply) pairs, never seen in training |
| Eval metric | Blind side-by-side review of generated text vs gold |
| Sampler | temp 0.7, top_p 0.9, repetition_penalty 1.15, repetition_context 64 (for both base and LoRA — apples-to-apples) |
| Eval harness | 4-column HTML viewer, label permutations rotated per row for blind testing |

## Methodological notes

### Why a sampler for both base and LoRA
Initial greedy-decode evaluation of the LoRA produced spurious repeat-loops ("user) user) user)..." 100 times) on otherwise plausible openings. This is a known failure mode of LoRA-shifted distributions under greedy decoding — the adapter biases certain tokens upward enough that they dominate top-1 selection. Adding `repetition_penalty=1.15` with `repetition_context_size=64` resolves the loops without changing the distribution shape.

Critically: **the same sampler must be applied to the base** for the comparison to be apples-to-apples. Greedy base + sampled LoRA would smuggle the LoRA an unfair quality advantage on length and diversity. Both base and LoRA generations in the published eval use identical sampler settings.

### Why mask-prompt is non-negotiable
The mlx_lm `--mask-prompt` flag computes loss only on the final assistant turn (the answer), not on the system prompt or user turn. Without this, the model partially learns to generate the system prompt and user message as well, which corrupts the personal-voice signal. The first iteration of training (without mask-prompt) showed loss climbing monotonically (5.5 → 7.2) due to a separate nested-template bug; the corrected version with proper masking gives smooth descent (5.31 → 4.16 val).

### Why 4-way, not 2-way
The minimum viable comparison is base vs LoRA. Adding full-FT lets us test whether LoRA's quality is bottlenecked by adapter capacity or by training data — if LoRA underperforms full-FT, the bottleneck is capacity (rank too small, layers too few); if LoRA matches full-FT, the bottleneck is upstream (data, methodology, eval). The observed result — parity — points at the second.

Adding the **ask** vs **reply** axis tests whether the personal voice recovers across distinct interaction modes (concise telegram reply vs verbose Claude Code request) or only on one. Both modes recover. This matters for the personal-AI design point because personal models that only work on one mode would be product-fatal.

## Implications

### 1. Personal-AI is a feasible deployment target
The 25× compression result moves personal models from "research curiosity" to "phone-class hardware deployable today". A 68 MB per-user delta is comparable to a single high-resolution photo. Sync, distribution, and storage costs are negligible. The deployment story for distributed personal AI no longer requires speculative compression breakthroughs.

### 2. Adapter-level personalization is a research-friendly unit
Because the adapter is small and the base is fixed, the "what does personal data do to a model" question has a tractable experimental setup: train a LoRA, measure the delta, run interpretability probes on the delta. This is much smaller than studying a frontier-scale RLHF model, and the unit (adapter) is the right scale for understanding personalization in isolation.

### 3. Methodological lesson: per-LoRA decode requires per-LoRA sampler tuning
Out-of-the-box greedy decoding with LoRAs produces spurious failures (repeat loops). Standard practice should be: tune sampler with the LoRA in place, and apply the same sampler to the baseline for comparison. This is a small but consequential gotcha for anyone doing LoRA-vs-base evals.

## What this connects to

- **[Era 10 finding](era10_matchbox.md)**: LoRAs install persistent priors. The 4-way Phase 0 result here is the text-domain analogue — a personal-LoRA installs a persistent style/voice prior that survives across reply and ask modes.
- **[Safety thesis](safety_thesis.md)**: makes personal models a feasible building block for the anti-concentration architecture (pillar 3, capability bounding).
- **Forward**: Phase 3 dyad-relational layer (in design) — when two personal LoRAs share state with each other, does the shared state exhibit properties neither personal model alone exhibited?

## Limits + honest disclosures

- **Eval is single-rater.** Blind side-by-side review by the model's owner (the only person who can ground-truth "did this sound like me"). Multi-rater inter-annotator agreement is the natural next step.
- **Repetition-penalty papers over distribution issues.** The fact that LoRA needs `repetition_penalty=1.15` to stop looping is a clue that the LoRA-shifted distribution has heavier mode peaks than the base. A cleaner methodology would diagnose and address this distribution-level rather than at the sampler.
- **Weak-language asymmetry observed.** On Russian prompts, the LoRA recovers surface markers (greetings, idioms, sentence-final particles) but content coherence is weaker than on English prompts. Most likely cause: Qwen3.5-0.8B has weaker Russian pretraining → larger LoRA-shift to recover Russian style → more drift in coherence. This is a real limitation, documented for honesty.
- **No public eval harness yet.** The eval harness is currently a one-off HTML viewer + JSON. Generalizing to a public benchmark is one of the proposed grant deliverables (see [safety_thesis.md](safety_thesis.md)).

## Reproducibility

Working code in private `pt-moe-research` repo (access on request). The eval methodology is straightforward to replicate on any personal corpus:

1. Extract (context, gold_reply) pairs from a personal conversation export
2. Hold out N pairs as eval set, blocklist them from training set
3. Train LoRA + Full-FT on remaining pairs with mask-prompt
4. Generate with matched sampler on the held-out pairs from all four variants
5. Render side-by-side HTML with label permutations rotated per row
6. Blind-rate

Total compute: ~30 min on M2 for the LoRA, ~3 hours for the full-FT.
