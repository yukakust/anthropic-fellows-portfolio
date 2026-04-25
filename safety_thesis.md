# Personal models as anti-concentration safety infrastructure

## Thesis

The dominant safety risk over the next decade is **concentration**: one or two foundation models serving a billion people, with values, failure modes, and political alignment all centralized in a small number of corporate weights. Behavioural safety work (RLHF, constitutional AI, evals) addresses *how* a centralized model behaves; it does not address *that* the model is centralized. The latter is a structural problem and requires structural mitigations.

The mitigation explored in this work is **shifting cognitive sovereignty back to individuals via personal models** — small adapters trained on an individual's data, sitting on top of (but not subordinate to) a shared backbone. Each user holds their own slice of the model. The backbone supplies broad capability; the personal adapter supplies values, voice, context, and agency.

This is not "AI for the user" in a product sense. It is **distributed AI ownership** as a structural safety property.

## Four pillars

### 1. Concentration containment
Distributed personal models bound the blast radius of any single failure. A misaligned centralized model affects every user simultaneously; a misaligned personal adapter affects one user, and the failure mode is observable to that user. The architecture replaces a single point of catastrophic failure with N points of small recoverable failure.

### 2. Interpretability tractability
Each personal LoRA is a *controlled mutation* of a known base. The delta between (base) and (base + LoRA) is small, bounded, and inspectable. Studying *how* personal data shifts behaviour — which circuits move, which features activate, which biases emerge — is a small interpretability subproblem with real ground truth (the user's own data and reactions). This is mechanistic interpretability work that does not require frontier-scale compute, because the unit of study is the adapter, not the base.

The Era 10 finding ([era10_matchbox.md](era10_matchbox.md)) directly supports this: a 49 MB LoRA acts as a persistent always-on domain prior, pulling every inference into its trained distribution regardless of trigger or prompt framing. This means personal LoRAs are *strong* interventions — a pixel-perfect lens for "what happens when we install personal data into a base model." Empirical, controlled, observable.

### 3. Capability bounding
A 0.8B-parameter model on-device, with selective backbone access, caps capability per node. The user retains full inference locally for routine tasks; backbone calls for harder tasks are gated. This is the opposite design point from "one model for everything" — it is a deliberate design choice to put a capability ceiling on each personal node, with the heavy compute (and heavy alignment burden) concentrated in the gated backbone calls.

### 4. Sovereignty
The user owns the weights. The personal adapter cannot be revoked by a platform. Training data does not leave the user's device. This is structurally different from "private mode" or "memory off" — it is an architecture where the user's slice of cognition is a file under their control, not a database row in someone else's account. Removing the adapter from the platform does not remove it from the user.

## Connection to existing work

| Component | What it shows | Status |
|---|---|---|
| [Phase 0 — Arrow routing](phase0_results.md) | Multiple LoRAs can be composed and routed at inference; learned capability vectors achieve 97% of oracle. Required if many personal/specialist adapters are routed per query. | Validated, math5 benchmark |
| [Era 10 — LoRA as persistent prior](era10_matchbox.md) | A LoRA is a strong always-on intervention, not a context-conditional switch. Validates the per-user adapter as a persistent personalization unit (and surfaces methodological pitfalls for evaluating style adapters). | Validated, 75+ blind votes |
| [Phase 0 — 4-way SFT eval (personal voice)](phase0_yuka_voice_eval.md) | A 68 MB LoRA recovers individual voice on telegram and engineering chat data at parity with full fine-tuning (1.7 GB). 25× compression. Phone-class hardware deployable. | Validated, 4-way blind eval |
| [Georgian-cuisine eval (125M specialist vs Claude)](georgian_cuisine_eval.md) | A 125 M model trained on 99 narrow-domain texts ties frontier Claude on direct domain retrieval (7-6-7 on 20 blind items). Two-specialist + router variant **beats** the same Claude 5-3. Operational evidence for pillar 1 (concentration containment): the user's personal model is *not worse than* the centralized model on the user's own narrow domains. | Validated, single-rater 20-item blind |
| **Phase 3 — Dyad-relational layer** | Personal models retain shared state with specific other people (federated, no central aggregation). Architectural delta vs any personal LLM today. | In design |

The Phase 3 dyad-relational layer is the forward research direction. To my knowledge, no one is exploring relational state at the personal-model layer — current systems treat the personal model as an isolated atom. A safety-relevant question this opens: when two personal models *share* state, does the shared state exhibit emergent properties that neither personal model alone exhibited? This is a small, tractable, safety-flavoured experiment that does not require a frontier model to run.

## Why this is research, not product

The product framing ("personal AI", "your own model") is shorthand for the underlying architecture. The architecture itself is a research bet:

- **Falsifiable**: each pillar above produces measurable claims (interpretability metrics on the LoRA delta, capability bounds, behavioural divergence between personal models, etc.).
- **Generalizable**: results from individual personal models inform the design of larger distributed-AI systems, regardless of whether the specific product (Jippy) succeeds commercially.
- **Cheap**: the unit of study is a sub-100 MB adapter on a 0.8B base. Experiments run on consumer GPUs in hours. The bottleneck is methodology, not compute.
- **Cumulative**: each personal model is a data point. As friends + collaborators run their own adapters, the dataset of "personal LoRA deltas + behavioural outcomes" grows naturally.

## What an external grant unlocks

Concrete, measurable outputs at the $30–60k scale:

1. **Interpretability writeup**: the LoRA delta against base, mechanistically characterized on a small controlled pair. Question form: *which circuits in the base model are most modified by N hours of personal-data fine-tuning?* Output: technical report + open-sourced eval harness.
2. **Phase 3 prototype**: dyad-relational layer working end-to-end on two paired personal models. Output: working code + design doc + demo.
3. **Phase 0 eval methodology**: the 4-way SFT eval (LoRA / Full-FT / Base / Reference) generalized to a public benchmark for personal-voice recovery. Output: open eval harness + leaderboard format.

All outputs published openly. No proprietary moat — the goal is to make the personal-model design point easier for other researchers to build on, not to lock in a single implementation.

## Contact

Yuka Kust · kustyuka@gmail.com · https://gpu.social
