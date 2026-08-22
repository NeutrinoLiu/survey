# N₀-TWAM — Scaling Tactile-Native World-Action Model for Contact-Rich Manipulation

**arXiv:2607.23783** · NeoteAI Team & Fudan TEAI Team · Jul 2026 · [code](https://github.com/neoteai/N0-TWAM) · [site](https://research.neoteai.com/n0-twam/)

**One line.** The first tactile world-action model trained at foundation scale, and the clearest articulation of the design axis the whole 2026 world-model cluster is arguing over: **should the model protect its visual stream from touch, or let them attend freely and separate them in weights instead?**

## 1. What "tactile" means here

Two representations, deliberately:

- **Predicted tactile** — a generated future contact stream, produced under the same flow-matching objective as future video.
- **Observed tactile** — the current reading, encoded through **NeoForce** (from N₀-Foundation).

NeoForce is the interesting piece. Its input is **not a raw gel image** but a dense three-axis surface force map `F̂ = g_θ(T) ∈ R^{H×W×3}` produced by a learned estimator:
- `[f_x, f_y]` — in-plane shear, which drives sliding and grasping
- `f_z` — normal pressure
- plus a **binary contact mask** → **six channels for a parallel gripper**

Because these units are *physical* rather than tied to a specific gel's appearance, one encoder serves different sensors. Visual and tactile streams are patchified independently and fused by a shared **ViT initialised from DINOv2**; a reconstruction head decodes the force map and contact mask back out, keeping the representation anchored to a physical quantity. `g_θ` stays **frozen** (a calibrated sensor-to-physics converter, gradients stop at the force map) while the NeoForce encoder is fine-tuned with the action loss.

## 2. Data curriculum

**NeoData: 30,000+ hours of vision–tactile data**, self-collected real-robot manipulation with **synchronised per-finger tactile**, spanning **six embodiments and 450 tasks**.

Also a **tactile-aware sub-task system**: tactile contact events (a grasp, a release) segment long-horizon demonstrations into sub-task-conditioned clips for training. At inference the **predicted** tactile triggers advancing to the next sub-task and the **observed** contact event confirms it — so both tactile roles drive the planner, not just the policy.

## 3. Model — asymmetric Mixture-of-Transformers, 7.16B

Built on a video-diffusion transformer (warm-started from LingBot-VA, a non-MoT shared-backbone world-action model) reorganised into **three per-modality experts** that interact **only through a single shared self-attention at every layer**:

| Expert | Hidden size dₘ | Params | Init |
|---|---|---|---|
| Video | 3072 | 5.00 B | warm-started from pretrained video WAM |
| Action | 1024 | 1.13 B | from scratch |
| Tactile | 1024 | 1.03 B | from scratch |
| **Total** | shared attn d = 3072 (24×128 heads) | **7.16 B** | (all-full-width ≈ 15 B) |

The asymmetry is justified explicitly: the action and tactile streams have **no large-scale pretraining to justify full width**, and the video expert is ~70% of the backbone, so slimming the two new modalities roughly halves the trainable model. Thin boundary projections leave the concatenated attention unchanged while FFN/norms/residual run at the narrower dₘ.

The factorisation:

```
p(x^v_{1:K}, x^t_{1:K}, x^a_{1:K} | c) = ∏_k p(x^v_k, x^t_k | X_{<k}, c) · p(x^a_k | x^v_k, x^t_k, X_{<k}, c)
                                              └──── predict ────┘   └──────── act ────────┘
```

Vision and touch are predicted **jointly at the same causal step**; the action is denoised conditioned on its own chunk's just-predicted video *and* tactile.

## 4. How tactile enters the model

This is the paper's central design claim, and it is worth quoting the position precisely. The authors catalogue three prior partial paths — tactile policies that consume touch and never predict it; frameworks with a separate frozen tactile predictor bolted on outside the acting network; and concurrent touch-aware world-action models that **"gate or mask tactile tokens to protect the visual stream"**.

N₀-TWAM's counter-position: **isolate capacity in weights, not attention.** Each modality gets private weights but vision and touch stay *fully attentive to one another* through one unmasked shared self-attention. Q/K/V from each expert are concatenated in fixed order `[v | a | t]` and run through one masked self-attention, where the mask encodes only causality, not modality protection.

The causal structure is a **frame-id diffusion-forcing cascade**: within a chunk, vision and touch tokens from the same latent frame share one position; the action follows the predicted frames it conditions on. Each token is noisy while being generated and clean once it is history. This turns shared attention into a "predict-then-act" cascade inside **one forward pass**.

Two more details worth noting. Language enters the **video and action experts via cross-attention but not the tactile expert** — touch is treated as physics, not semantics. And the **observed** pathway is a lightweight side branch that cross-attends current tactile into action tokens before the action head, so at inference, once a chunk's video and tactile are predicted, their attention K/V are cached and each action-denoising step re-runs **only the slim action expert**.

## 5. Experiment setup

- **Simulation** — UniVTAC (8 tasks), NeoSim (4 single-arm + 8 dual-arm)
- **Real** — eight contact-rich tasks on two embodiments: four single-arm on a Flexiv, four dual-arm on a PiPER; six of eight drawn from NeoReal
- **Baselines** — π₀.₅ (VLA), LingBot-VA (vision-only WAM), and other vision-only WAMs
- **Generalisation axes**, each isolated on the task where it matters: unseen object (Bottle Standing, Bag Packing), unseen position (Cup Stacking), visual perturbation (Fruit Collection — lighting/background change)

## 6. Does tactile actually help?

**Headline.** UniVTAC **84.5%**, NeoSim **49.4%**, real-robot average **46.3%** — against **30.0%** for the strongest VLA baseline and **21.9% / 14.4%** for vision-only world-action baselines.

**Ablations** (task-averaged success %):

| Variant | UniVTAC | NeoSim | Mean |
|---|---|---|---|
| **Full** | **84.5** | **49.4** | **67.0** |
| w/o predicted tactile | 71.8 | 41.1 | 56.4 |
| w/o observed tactile | 70.5 | **29.6** | 50.0 |
| 20% pre-training data | 65.4 | — | — |

Two things stand out. First, **pre-training scale is the single largest factor on UniVTAC** — a 20% checkpoint costs 19.1 points, more than either tactile pathway. That is an unusually honest thing to publish in a paper whose contribution is the tactile architecture. Second, both pathways are genuinely load-bearing, and the ordering *flips* between benchmarks: on NeoSim, removing **observed** tactile is far more damaging (−19.8) than removing predicted (−8.3), while on UniVTAC they are nearly equal.

The ablation methodology deserves credit. Deleting the predicted pathway would change the sequence layout, so instead they apply an **information ablation** — zeroing the predicted tactile latent *before* it is noised, leaving sequence layout, attention mask and diffusion loss token-for-token identical. Only the information changes.

**Generalisation** (real-robot success %):

| Method | Unseen object | Unseen position | Visual perturbation | Avg |
|---|---|---|---|---|
| π₀.₅ | **80** | 45 | 25 | 50.0 |
| LingBot-VA | 65 | 35 | 30 | 46.7 |
| **N₀-TWAM** | 75 | 45 | **45** | **51.7** |

This table is the most useful in the paper because it says where touch *doesn't* help. On **unseen objects π₀.₅ wins outright (80 vs 75)** — large vision–language pretraining carries object and semantic priors that world-action models, leaning on learned dynamics, do not have. Tactile buys nothing for novel appearance. The decisive axis is **visual perturbation**, where π₀.₅ and LingBot-VA fall to 25 and 30 while N₀-TWAM holds at 45. The authors state the conclusion correctly: not strongest on every axis, but **most robust**, degrading most gracefully exactly where vision becomes unreliable.

**Sim-to-real signal.** Reading gel-rendered simulated tactile through the real-pretrained NeoForce encoder instead of a from-scratch encoder raises UniVTAC from 82.4 → **88.1** and single-arm NeoSim 58.5 → **64.8**. Gains concentrate on tasks turning on weight-bearing or shear cues — Put Bottle in Shelf 82 → 96, Insert HDMI 63 → 75, Pull-out Key 74 → 83, Grasp Chip 82 → 92, Pour Ball 55 → 68 — while sharp-contact insertion tasks near ceiling barely move.

**Stated future work.** Faster streaming decoding; a longer predicted horizon; and **broader sensor coverage** — the learned tactile representation is currently trained on a narrow set of sensor types.

## 7. What it adds that the others don't

The scale, and the architectural argument. It is the only tactile world model with a published data-scaling ablation, and the only one to make the **weights-vs-attention** case explicitly against the gating approach of [[dream-tac]] and the asymmetric attention of [[tactile-wam]]. The force-space representation (NeoForce) is also the strongest answer in this survey to the cross-sensor problem [[tacverse]] measures: normalise to *physical units* rather than to a learned token space, and one encoder serves many gels. Compare [[vt-wm]], which reaches similar conclusions at ~1/1000 the data scale with frozen encoders and no gating either.
