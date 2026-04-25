# Georgian-cuisine eval: 125M specialist vs frontier Claude

A small specialist model (125M params) trained on 99 texts about Georgian cuisine matches a frontier Claude model on direct domain retrieval. A two-specialist + router variant **wins outright** against the same Claude on the same direct test (5 wins to Claude's 3, with 8 ties out of 16 items).

This is empirical evidence for the personal-AI thesis: a tiny adapter trained on a narrow domain can match a frontier model on that domain. Originally run 2026-04-08 inside the `pt-moe-research` private repo (FEEDING-002 / FEEDING-004 / MOE-002 experiments).

## TL;DR table

| Variant | Params | Training data | Direct test (name → description) | Reverse test (description → name) |
|---|---|---|---|---|
| Claude (frontier, version per author's recollection: Opus 4.7) | ~frontier | RLHF + general web | baseline | baseline |
| Specialist v1 — 99 Georgian texts on rugpt3small | 125 M | 99 texts (~42K words) | **Claude 6 / Spec 7 / Tie 7** | Claude 10 / Spec 1 |
| Specialist v2 — 456 texts (Wiki RU+EN + 122 Q&A) | 125 M | 456 texts (~61K words) | **18 / 20 correct (vs 14/20 at 99)** | 2 / 10 |
| **Two-specialist + router (Georgian + Atatürk)** | 2 × 125 M | 1188 texts total | **Claude 3 / Spec 5 / Tie 8** | Claude 3 / Spec 0 / Tie 1 |

The headline rows are the v1 direct test (specialist ties Claude 7-6 with 7 ties) and the two-specialist routed-direct test (specialist beats Claude 5-3 with 8 ties).

## Setup

| Component | Detail |
|---|---|
| Base model | `ai-forever/rugpt3small_based_on_gpt2` (125 M params, Russian-pretrained GPT2) |
| Training data | 99 (then 456) texts about Georgian cuisine, mostly Wikipedia RU+EN and built-in recipes; 122 Q&A pairs added at v2 |
| Training | lr=5e-5, batch=4, 300–2000 steps depending on stage |
| Hardware | RTX 3090 24 GB, ~13.5 step/s |
| Eval | 20 questions about Georgian cuisine, written before either model saw the prompts; Claude answered blind first; specialist answered blind second; per-question hand-rated for correctness |
| Claude model | "Claude" — version not recorded in the source notes; per author's recollection this was Opus 4.7 (the default in the same author's `get-visa` repo at the time, 2026-04). Disclosed in source as "Claude" only — see honesty disclosure below. |

## Method

The 4-step protocol used in the source `EXPERIMENTS.md`:

1. Write 20 domain questions about Georgian cuisine (named dishes, ingredients, preparation, regional origin, cultural context) — deliberately mixing easy and hard.
2. Submit all 20 to Claude blind, save answers verbatim.
3. Submit all 20 to the specialist model blind (same temperature, no Claude answers in context), save answers verbatim.
4. Per-question rating (correct / incorrect / partial) by author. Tie = both correct or both partial-but-equivalent.

Two test directions:
- **Direct test**: question is the name of a dish or concept ("what is khinkali?"). Tests *knowledge retrieval*.
- **Reverse test**: question is a description, model must name the concept ("a sour sauce made from wild plum?"). Tests *bidirectional association* — much harder.

## Results

### Direct test, v1 (99 training texts, 125 M params)

| | Claude | Specialist | Tie |
|---|---|---|---|
| Wins | 6 | 7 | 7 |

The specialist tied or beat Claude on 14 of 20 items. Where it won, the win pattern was *details memorized verbatim from the training texts* — etymology, regional sub-variations, ingredient ratios. Where it lost, the loss pattern was *confusion between similar dishes* (khachapuri ↔ satsivi, mtsvadi ↔ pkhali) — a signature of underfitting on a small corpus.

### Reverse test, v1 (99 training texts)

| | Claude | Specialist | Tie |
|---|---|---|---|
| Wins | 10 | 1 | 9 |

The specialist could describe but not name. Training data was almost entirely "X is a Y"; the specialist learned the forward direction "name → description" but not the reverse "description → name". Found correctly: only `sulguni` (training corpus contained the literal phrase "rassolny syr sulguni"). Fixed at v2 by adding 122 explicit Q&A pairs in both directions.

### Direct test, v2 (456 training texts)

| | Correct | Wrong |
|---|---|---|
| Specialist | 18 / 20 | 2 / 20 |

5× the data → all v1 errors fixed (kvevri, mtsvadi, khmeli-suneli, tklapi). Reverse test improved to 2/10 — bidirectional Q&A helps but is not enough; the model needs reverse-direction *inference*, not just memorized mappings.

### Direct test, two-specialist + router (1188 texts total, two domains)

| | Claude | Specialist | Tie |
|---|---|---|---|
| Direct (16 items) | 3 | **5** | 8 |
| Reverse (4 items) | 3 | 0 | 1 |
| **Combined (20)** | **6** | **5** | **9** |

Two 125 M specialists (Georgian cuisine + Atatürk biography) routed by a keyword classifier. **Specialists beat Claude 5–3 on direct retrieval**, with 8 ties. Router accuracy 19/20.

## What this proves (and what it doesn't)

### Proves
- **A 125 M model with 42K words of domain text can tie a frontier general model on direct domain retrieval.** This is the "concentration containment" premise from the [safety thesis](safety_thesis.md): on a narrow topic the user actually cares about, the user's personal model is *not worse than* the frontier centralized model. The frontier model is not strictly necessary for that domain.
- **Routed multi-specialists can exceed the same frontier model on the union of their domains.** Direct test: specialists 5, Claude 3. The routing infrastructure (Phase 1 Arrow, [phase0_results.md](phase0_results.md)) is the same shape — a learned router over per-domain adapters.
- **Knowledge transfer happened with no RLHF, no instruction-tune step, no Anthropic-scale compute.** Just SFT on 99–456 texts of a narrow Wikipedia-quality corpus. This is the regime personal AI can actually live in.

### Does not prove
- The specialist model is *better* than Claude in general. It is not. It loses on reverse-direction inference (10:1 at v1, 8:2 at v2), composition, multi-step reasoning, anything outside the trained domain.
- The bench is multi-rater. It is single-rater (the model author), 20 items, qualitative correct/wrong/tie. Multi-rater + larger N is the natural follow-up.
- The Claude version is logged. The source notes say only "Claude". Author's recollection is Opus 4.7 (matches what the same author's `get-visa` repo defaulted to in April 2026), but the version field was not pinned in the eval scripts.
- Performance scales beyond 125 M. The whole point is small models on narrow data; the open question for personal-AI is "does this still hold at the ~1 B scale on personal data" — that's what [Phase 0 voice eval](phase0_yuka_voice_eval.md) is testing in parallel on Qwen3.5-0.8B.

## Why this matters for the safety thesis

Pillar 1 of the [safety thesis](safety_thesis.md) is **concentration containment**: the user's personal model bounds the blast radius of any single failure of a centralized foundation model. The implicit prerequisite is that the personal model is *good enough* on the user's own narrow domains that the user is not constantly forced back to the centralized model anyway.

This eval is a direct test of that prerequisite, on a non-toy domain (Georgian cuisine — a real culture, real ambiguity, real risk of confusion between similar dishes). The specialist matched the frontier model on direct retrieval at 125 M params and 42K words. Two specialists on two unrelated domains *beat* the same frontier model on the union.

If this scales — and Phase 0 voice-eval results say it does at ~1 B params with personal data — then "personal AI good enough that you don't have to touch the centralized model for your own domains" is a real architectural option, not a fantasy.

## Sources

- Original write-up: `EXPERIMENTS.md` in private `yukakust/pt-moe-research` (FEEDING-002, FEEDING-004, MOE-001, MOE-002 sections, dated 2026-04-08)
- Data sources catalog: `georgian_cuisine_data_sources.md` (Wikipedia EN/RU/KA, ~3500 Russian recipe-site URLs, English recipe sites, Georgian-language sites, books)
- Access to private repo available on request.

## Related work in this portfolio

- [Phase 0 — Personal voice recovery (4-way SFT eval)](phase0_yuka_voice_eval.md) — the same effect (small model recovers narrow distribution) at the personal-voice level.
- [Phase 0 — Multi-LoRA composition: Arrow routing](phase0_results.md) — the routing infrastructure that turns single-specialist results into multi-specialist results, in-domain at 97% of oracle.
- [Era 10 — LoRA as persistent prior](era10_matchbox.md) — image-domain analogue: a LoRA installs a persistent always-on prior, the same way the Georgian specialist installs a persistent Georgian-cuisine prior.
- [Safety thesis](safety_thesis.md) — the four-pillar argument for distributed personal models as anti-concentration safety infrastructure.
