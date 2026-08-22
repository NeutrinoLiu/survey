# TacForeSight — Force-Guided Tactile World Model for Contact-Rich Manipulation

**arXiv:2606.11184** · TARS Robotics + NUS + SJTU + CASIA + Fudan (Zang, Y. Zheng, Nie, Y. Zheng, Tian, Gu, Gao, Z. Wang, Yan, Ding) · Jun 2026 · [site](https://tacforesight.github.io/ProjectPage)

**One line.** Built on one physical observation — **wrist wrench changes precede fingertip tactile responses** — and turns that lead time into a force-conditioned predictor of future tactile latents, running at 20 Hz.

## 1. What "tactile" means here — two signals with asymmetric roles

- **Dual-finger optical tactile** — fine-grained *local* deformation fields, encoded as 3D marker-displacement channels
- **Wrist 6-axis force/torque** — high-frequency *global* cues about external load change

The central claim is a **coarse-to-fine temporal dependency**: in bulb insertion and locking, abrupt wrist-wrench changes *consistently precede* fingertip tactile responses. Global force is a **leading indicator of future tactile state**. The authors ground this in human sensorimotor control literature (Johansson & Flanagan) — load-related cues are used to anticipate interaction states and guide local contact adjustment.

Their critique of prior work follows directly: existing methods "rarely model the asymmetric spatiotemporal roles of global force and local tactile sensing," treating them either as two modalities to fuse into a joint representation or as passive reactive feedback. Both are symmetric treatments of an asymmetric relationship.

## 2. Data curriculum

Five real-world contact-rich tasks — **Bulb Insertion & Locking, Tube Adjustment & Insertion, Vase Wiping, Card Swiping, Wire Insertion** — plus **three in-process perturbation settings** that deliberately disrupt ongoing contact states mid-execution.

## 3. Model — two cascaded stages

**Stage 1 — TacForceWM** (force-conditioned tactile world model):
- Tactile tokenizer per finger + a **FingerID** embedding → `F^s_{t−H:t}`
- Self-attention → fused tactile feature `z^tac_{t−H:t}`
- **Force encoder** on the wrist wrench → `c_{t−H:t}`
- Force conditioning enters through **AdaLN** modulation of the predictor (× N blocks), plus positional encoding
- Output: **predicted tactile latents** `ẑ^tac_{t−H+Δ : t+Δ}`, supervised by a **latent loss**

**Stage 2 — Predictive Tactile-Conditioned Policy**:
- DINOv2 spatial encoder for image observations → `h^img_t`
- **Cross-attention** between current tactile `z^tac` and predicted tactile `ẑ^tac` — modelling current→future tactile evolution
- **Tactile-guided gate** adaptively fusing visuo-tactile features
- Flow-matching action head → action chunk

## 4. How tactile enters the model

Three distinct entry points, in a specific order:

1. **Force conditions tactile prediction**, not the other way round — the asymmetry is built into the architecture via AdaLN force modulation of the tactile predictor.
2. **Predicted tactile latents are anticipatory contact priors** for the policy, delivered by cross-attention against current tactile.
3. **Tactile-guided gating** weights visual vs. tactile features.

The key efficiency decision: forecasting happens **purely in a compact latent space** — no raw tactile images, no pixel-space video generation. The authors contrast this explicitly with [[vt-wm]] and [[omnivta]], where generating high-dimensional sensory observations is "computationally expensive for real-time control," and with representation-learning approaches (exUMI, DreamTacVLA) that use tactile prediction only as an **auxiliary objective** and never use the prediction at execution time.

Result: **20 Hz real-time inference on a single RTX 4090D**.

## 5. Experiment setup

Five tasks × {nominal, three perturbation regimes}. Perturbations are *in-process* — they disrupt contact that is already established, which is the regime where anticipation should matter and reaction should fail.

## 6. Does it work?

TacForeSight achieves state-of-the-art across both nominal and perturbation settings, **with the margin largest under dynamic contact disturbances**. The reported gains decompose into three capabilities the perturbation design is built to separate: **contact establishment, contact maintenance, and recovery from dynamic contact disturbance**.

That pattern is the expected signature of anticipation rather than better perception. A reactive policy and a predictive one should look similar under nominal conditions and diverge when contact changes faster than the control loop can respond — which is what the perturbation results show.

## 7. What it adds that the others don't

The **force→tactile temporal ordering** as an architectural commitment. Every other work in this cluster treats wrist wrench and fingertip tactile as two members of a "contact" modality group to be fused; TacForeSight uses one to *predict* the other, on the evidence that they are separated in time. It is also the lightest predictive design here — latent-only forecasting at 20 Hz, versus video-diffusion backbones — which makes it the practical counterpoint to [[n0-twam]] and [[dream-tac]]: you do not need to generate pixels to get useful foresight, if you are willing to give up the visual future entirely (a conclusion [[omnivta]] reaches independently by measurement).
