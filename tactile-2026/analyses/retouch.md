# ReTouch — Empowering Contact-Rich Dexterous Manipulation with Online-Refined Tactile Prediction

**arXiv:2608.01824** (v2) · USTC + iFLYTEK + CUHK (S. Zhang, X. Zhang, Shen, Y. Li, Y. Gao, S. Zhang, Y. Zhang, Long, J. Wu, J. Pan, Deng, Y. Zhang) · Aug 2026

**One line.** Observes that a tactile prediction made at the start of an action chunk is **stale by the middle of it**, and refines the prediction *inside* the chunk — worth 15.2 points over keeping it fixed.

## 1. Two problems with tactile in dexterous VLAs

**(a) Structure is destroyed by flattening.** *"Dexterous hands produce dense tactile observations, while task-relevant contacts are typically sparse and localized to specific fingers and fingertip regions. Existing methods typically flatten tactile signals into generic tactile tokens or compress each finger's temporal force history into representations. Such representations obscure the finger-wise spatial structure of contact, making it difficult for the policy to associate local contact changes with fine-grained finger adjustments."*

**(b) Predictions go stale.** *"Contact states in multi-finger manipulation can change rapidly because of slippage, contact displacement, and execution errors, causing future tactile predictions generated at the beginning of an action chunk to become stale during execution."* Existing predictive methods do not update them; existing predictive-reactive methods refine the *action* but not the *prediction guiding it*.

The paper also gives the clearest taxonomy of the tactile-policy literature: **direct-fusion** → **reactive** → **predictive** → **predictive-reactive**.

## 2. Model

**Tactile-Patch Encoder** — structured tactile patch features *"that preserve finger identity and local contact structure"*, rather than a flattened token soup.

**High-frequency action module** — jointly predicts future tactile states **and** action chunks, then **refines both using incoming tactile feedback during execution**, with a blocking update so refinements take effect immediately.

## 3. Data

**XHT-Dataset** — 900 real-world demonstrations across seven contact-rich tasks on an **XHand–UR7e** platform.

## 4. Results

**+18.4** points over the strongest baseline under standard conditions and **+23.8** under challenging conditions.

**Ablation (average success):**

| Variant | Success | Δ from full |
|---|---|---|
| **Full ReTouch** | **83.6%** | — |
| Non-blocking refinement (parallel, ~1 step late) | 75.8% | **−7.8** |
| Refine actions but **fix** initial tactile prediction | 68.4% | **−15.2** |
| **No** future tactile prediction at all | 67.6% | −16.0 |
| w/o Tactile-Patch Encoder (flatten + MLP) | 69.1% | −14.5 |
| No refinement of either predictions or actions | 60.0% | **−23.6** |

Two findings, both sharp:

**A fixed one-shot tactile prediction is worth almost nothing.** 68.4% with a frozen prediction versus 67.6% with no prediction at all — a **0.8-point** difference. *"The main gain comes from continually refining future tactile predictions with the latest feedback."* That is a pointed result for the predictive-tactile cluster ([[tacforesight]], [[dream-tac]], [[n0-vtla]], [[hitac-wam]]), most of which forecast once per chunk.

**Timing matters at the single-step scale.** The non-blocking variant, where refinements land roughly one action step late, loses 7.8 points.

And the structural encoding is worth 14.5 points on its own, supporting the finger-identity argument.

## 5. The conclusion, well put

> *"Tactile prediction is therefore most effective as **a control state updated by execution feedback rather than a fixed forecast**."*

## 6. What it adds that the others don't

**Intra-chunk refinement of the prediction itself**, and the ablation isolating it from action refinement. This is the sharpest empirical challenge in the survey to the "predict the tactile future, then act on it" pattern: the prediction is only useful while it is fresh, and freshness is measured in single action steps. Combined with the Tactile-Patch Encoder's finger-identity preservation, it makes a strong case that dexterous tactile is not a signal to be pooled but a spatially structured state to be tracked.
