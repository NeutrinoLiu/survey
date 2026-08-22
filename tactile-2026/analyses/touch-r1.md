# Touch-R1 — Reinforcing Touch Reasoning in MLLMs

**arXiv:2605.27154** · Xiamen University + Great Bay University + Fudan + Nanjing + **Daimon Robotics** (Lai, Y. Zhou, F. Zhu, S. Zhu, W. Yuan) · May 2026

**One line.** Brings the R1 paradigm to touch, and the reward design is the contribution: **credit is assigned only when real tactile input beats counterfactual controls** where the tactile stream is removed, shuffled, or noise-masked.

## 1. The failure it targets

*"In visual-tactile conflict settings, models such as GPT-4o and Gemini-2.5-Pro tend to follow the visual object prior rather than revise their predictions from tactile evidence."* Existing tactile-language models (Octopi, TVL, Touch100k, AnyTouch, SToLa, VitaTouch) *"do not verify whether the final answer is grounded in tactile evidence"* — so they may learn visually plausible, object-category-driven answers without learning **when tactile evidence should override vision**.

## 2. Why vanilla R1 fails on touch — three modality-specific obstacles

1. **No pretrained prior.** *"Marker displacement, elastomer deformation, and contact-induced texture patterns are largely absent from natural-image pretraining corpora."*
2. **Cross-sensor shift.** GelSight Mini, Xense, Tac3D, DM-Tac X differ in resolution, marker layout, illumination and optical geometry, *"making pixel-level alignment across sensors unreliable."*
3. **Physical attributes are ordinal, not categorical.** *"Predicting moderate hardness for a soft object is physically closer than predicting hard, but sparse binary rewards treat both mistakes equally."*

## 3. Data and model

**TouchReason-1M** — >1M synchronised tactile pairs across **four optical sensors**, with verified reasoning-style QA annotations. **TouchReason-Bench** evaluates tactile perception *and* visual-tactile conflict resolution.

**Three training stages** on Qwen2.5-VL-7B:
1. **Tactile dynamics pretraining** — a ViT tactile encoder trained to predict the **latent code of the next tactile frame**, encouraging sensitivity to deformation dynamics rather than static appearance.
2. **QA supervised fine-tuning** (cold start).
3. **Tactile-grounded GRPO** with four reward components:
   - **ordinal-aware accuracy** — distance-aware, so near-misses are credited
   - **cross-sensor physical consistency**
   - **structured-format control** — preserving a parsable `<perceive> / <compare> / <conclude>` interface
   - **input-side tactile grounding** — the **tactile-use reward**, which *"assigns credit only when authentic tactile inputs yield superior correctness relative to counterfactual controls where the tactile stream is removed, shuffled, or noise-masked."*

That last reward is the paper's real idea: it makes "did touch actually cause this answer?" a trainable objective, and it is exactly the causal check missing from the rest of the tactile-language literature.

## 4. Results

**TouchReason-Bench average:** Touch-R1-7B **60.1** vs SToLa 47.5, Octopi-13B 41.7, Gemini-2.5-Pro 37.6, GPT-4o 35.4, Qwen2.5-VL-7B 28.0. **+18.4% over Octopi-13B, +24.7% over GPT-4o, +22.5 points over Gemini-2.5-Pro.** Gains concentrate on structured-reasoning metrics: **CSC +19.4**, **L2-EM +17.4** over SToLa, with **OMAE dropping 0.45 → 0.24** (incorrect predictions land closer to truth on the ordinal scale).

**VTV150K** (external benchmark), average: Touch-R1 **65.5** vs VTV-LLM-7B 60.4, Gemini-2.5-Pro 29.5, GPT-4o 28.0, Qwen2.5-VL-7B 20.6. The *Combined* attribute column is telling — GPT-4o scores **2.1** and most open models **0.0–2.1**, versus 41.9 for Touch-R1. Multi-attribute tactile reasoning is essentially absent from general MLLMs.

**Ablation** — each reward fixes a distinct failure:

| Stage | Avg | Effect |
|---|---|---|
| Qwen2.5-VL-7B base | 28.0 | — |
| + cold-start SFT | 47.8 | teaches the structured format — largest single boost |
| + standard GRPO | 50.1 | *"final-answer supervision alone cannot ensure reliable tactile-grounded reasoning"* |
| + ordinal reward | — | OMAE 0.42 → **0.31** (mitigates ordinal collapse) |
| + consistency reward | — | CSC 56.1 → **69.2** (largest cross-sensor gain) |
| + format reward | — | smaller but stable |

The +2.3 from vanilla GRPO versus the large gains from the tactile-specific rewards is the paper's argument in one row.

**Scaling** improves L2-EM, SOI-ρ, CSC and Avg consistently with backbone size.

**Emergent behaviour:** structured traces show **probing, comparison and revision**. The worked example is a good one — a visual photo suggests plastic; the tactile depth response grows slowly under increasing normal force; the model reasons that a rigid object would transfer stronger deformation to the elastomer, so the object must be absorbing it, and revises to **rubber**. GPT-4o, Gemini and Qwen all answer plastic.

## 5. What it adds that the others don't

The **tactile-use reward** — the only mechanism in this survey that trains a model to be *causally* dependent on touch by comparing against ablated-input counterfactuals. Combined with **ordinal-aware credit** (physical attributes are continuous, and binary rewards throw that away), it is a reward-design template that transfers to any physically grounded multimodal reasoning problem. See [[touchthinker]] for the complementary attack on the same problem from the data and representation side.
