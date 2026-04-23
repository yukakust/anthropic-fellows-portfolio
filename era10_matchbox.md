# Era 10 — Matchbox LoRA: LoRA as a persistent domain prior

## TL;DR

A LoRA fine-tuned on 500 Wikimedia Commons matchbox labels does not behave like a context-conditional switch keyed on a trigger word. It behaves like a **persistent always-on domain prior** baked into the weights: every inference is pulled into the LoRA's domain regardless of prompt framing or trigger inclusion.

Replicated across 75+ blind votes in 3 question-framing rounds. The same images can win 47% or 80% depending only on what the human is asked to evaluate.

---

## Setup

| Component | Detail |
|---|---|
| Base model | FLUX.2 |
| Dataset | 500 Wikimedia Commons matchbox labels, 11 categories, captions parsed from titles + trigger token `jppy-matchbox` |
| LoRA config | r=32, α=64 (baked scale=2.0), wide_modules (7 attention + 4 FF = 11 modules), 3000 steps |
| Training compute | ~17.5 min on RTX 4090 |
| Adapter size | 49 MB |

## Eval design

Three independent blind-vote rounds across the **same** 15 image pairs:

| Round | Prompt framing | Vote question | Base wins | LoRA wins | LoRA win % |
|---|---|---|---|---|---|
| `era10_matchbox` | descriptive on both | "matches prompt" | 5 | 10 | 64% |
| `era10b_strict` | bare-subject on base, trigger-only on LoRA | "matches prompt" | 8 | 7 | 47% |
| `era10b_style` | same as strict | "looks more like matchbox?" | 3 | 12 | **80%** |

## Core finding

> **The vote flipped from 47% → 80% just by rephrasing the question from "matches prompt" to "looks more like matchbox style."**
> Same images. Same LoRA. Different question. Different answer.

The bare prompt `factory worker` — no mention of matchbox, vintage, retro, or any domain word — renders as:

- **Base FLUX.2:** a normal-looking photograph of a worker
- **+ matchbox LoRA:** a retro Soviet matchbox-label illustration of a worker

The trigger token is unnecessary. The prompt context is unnecessary. The LoRA pulls every inference into its domain regardless.

## Implications

### 1. LoRA = persistent always-on domain prior

This is empirically demonstrated rather than assumed. The LoRA is not a switch toggled by `jppy-matchbox`; it is a prior baked into the weights that biases every generation toward the matchbox-label distribution. Trigger and context are not required for the prior to manifest.

### 2. Personalization-by-fine-tuning is a different design point than personalization-by-context

This validates an architectural premise for personal-AI systems: per-user style/domain/preferences can live in adapter weights rather than in retrieved context or system prompts. A user does not need to repeatedly say "I prefer X" — the adapter pulls inference into the user's domain automatically. The trade-off is the adapter cannot be turned off cleanly without scale control or routing.

### 3. Methodology: style-LoRA evals require style-aware questions

Content-matching question framings systematically punish style-focused LoRAs because the baseline's "more accurate to the bare words" renders win on prompt-content matching. **A style-LoRA evaluation must use a style-aware vote question** to surface what the LoRA actually does. Otherwise the methodology measures the wrong axis.

### 4. Limitation — symmetric to (1)

The same property that makes the LoRA work (always-on prior) also means it biases out-of-domain requests. Mitigations:

- Inference-time scale control (reduce LoRA strength for OOD requests)
- Multi-LoRA Arrow-style routing for when-to-engage decisions (see [Phase 0 results](phase0_results.md))

## Reproducibility notes

- Dataset: Wikimedia Commons category scrape with title-as-caption + manual category-trigger token
- Training: standard PEFT LoRA on FLUX.2 base, hyperparameters as table above
- Eval harness: pairwise A/B vote server, 5+ blind voters per round, vote DB stored per-experiment
- 75+ total votes across 3 rounds, results re-aggregated per question framing

Working code in private `pt-moe-research` repo (access on request).

## What this connects to

The persistent-prior finding is consistent with the Phase 0 multi-LoRA composition results (Arrow routing achieves 97% of oracle in-domain) — both point at LoRAs as domain-installable units that can be composed and routed at inference time. The methodological note (style-aware framings) generalizes beyond style: any LoRA-as-installation deserves circuit-level scrutiny rather than only behavioral evaluation.

A natural follow-up question: **where in the network does the persistent prior live, and through what circuits does it dominate prompt-conditional generation?** Mechanistic-interpretability probes (SAE features, activation patching with toggled adapters) on a controlled small-scale text-LoRA pair would localize the answer.
