# Phase 0 — Multi-LoRA composition: Arrow routing validation

Empirical validation of Arrow routing as a multi-LoRA composition method on a controlled math benchmark.

## TL;DR

**Arrow routing achieves 97% of oracle-routing accuracy in-domain** on a 5-LoRA math5 benchmark — strong existence proof that learned capability vectors can route queries to the correct LoRA without ground-truth labels. Production multi-LoRA serving (per-user, per-specialist, per-domain) requires this kind of label-free routing.

---

## Setup

| Component | Detail |
|---|---|
| Base | Qwen / SmolLM family |
| Adapters | 5 math-domain LoRAs (math5 benchmark — algebra, geometry, calc, etc.) |
| Routing methods | Oracle (knows ground-truth domain) · Arrow (learned capability vectors) · Random |
| Eval | Generation-jaccard on math5 split |

## Headline result

**Arrow ≈ 97% × Oracle on in-domain queries.**

This is the strongest signal that learned capability vectors can route queries to the correct LoRA without ground-truth labels — a prerequisite for production multi-LoRA serving where you cannot rely on a ground-truth domain labeler.

## Methodological lessons

### Lesson 1: loss-eval misled us

Initial development used loss-on-validation as the eval metric. Loss-based eval gave the wrong answer about which routing methods worked — it ranked methods by smoothness of likelihood rather than by usefulness of generation. Switching to **generation-jaccard** (overlap on actual generated tokens vs. reference) revealed the true ranking and the 97% Arrow result.

**Generalization:** for routing/composition methods, prefer generation-quality evals over loss. Loss can move in the wrong direction relative to generation quality, especially when the routing decision is upstream of token-by-token autoregression.

### Lesson 2: format mimicry → Layer 0 hypothesis

Cross-domain LoRA application (applying a math LoRA to a non-math query, or vice versa) produced output that took on the **format** (math notation, problem structure, equation layout) of the active LoRA's domain even when the content was wrong.

This is consistent with the hypothesis that early-layer (Layer 0 / embedding-adjacent) representations carry domain-format signal that is detectable via Arrow's capability-vector approach. The format-mimicry observation was unplanned but reproducible — it surfaced naturally during ablation runs and is the kind of artifact that a circuit-level account would explain.

## Phase 1 — hierarchical routing (in progress)

Building on Phase 0:

- **2-level Arrow routing:** first to a LoRA "guild" (cluster of related adapters via embedding clustering), then within the guild to the specific LoRA
- **Goal:** make Arrow tractable when the adapter zoo grows from 5 → 100s → 1000s. Flat Arrow scales poorly past O(100) adapters; hierarchical guilds restore scalability
- **Parallel medical benchmark:** 8 knowledge-injection methods compared on MedQA (LoRA, DCD, K-Adapter, MemDec, UniR, etc.) — testing whether the routing-based composition story holds up on a knowledge-heavy domain rather than a math-skill domain

## Why this matters

Production multi-LoRA serving (per-user LoRAs, per-specialist LoRAs, per-domain expert LoRAs) requires routing that scales without ground-truth oracles and without making the routing decision more expensive than the inference itself. Phase 0's 97% result is the existence proof for this approach at the in-domain limit. Phase 1's hierarchical routing is the scalability story.

For an end-user system (e.g. a personal-AI product like [Jippy](https://gpu.social/jippy/)) this enables: a backbone model + N tiny user-specific or specialist LoRAs, with a small router that decides which (or which combination) of LoRAs to apply per query — without per-query routing decisions becoming the bottleneck.

## What this connects to

- **Era 10 finding** ([writeup](era10_matchbox.md)): demonstrates that LoRAs install persistent domain priors. Combined with Phase 0's routing result, this gives a coherent story: composable LoRAs as installable, routable domain priors at inference time.
- **TRAINING_RULES framework** (pt-moe-research): standardizes base whitelist, eval harness, capability-vector extraction, and the contribution-economy royalty scheme for forks built on top of the routing infrastructure.

Working code in private `pt-moe-research` repo (access on request).
