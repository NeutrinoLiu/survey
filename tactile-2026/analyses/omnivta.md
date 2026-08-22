# OmniVTA — Visuo-Tactile World Modeling for Contact-Rich Robotic Manipulation

**arXiv:2603.19201** (v3, Aug 2026) · NUS + CASIA + Fudan + TARS Robotics + Tsinghua + Zhongguancun Academy + Beihang (Y. Zheng, Gu, Y. Zheng, … Yan, Ding) · Mar 2026

**One line.** A large visuo-tactile dataset (**21,879 trajectories**) plus a slow-fast system whose most interesting result is negative: **generated *visual* futures don't help the policy**, only generated *tactile* futures do.

## 1. What "tactile" means here

Multi-sensor by design — the collection platform supports **Xense, GelSight Mini, Tac3D, and Daimon** on identical end-effectors. Tactile is encoded by **TactileVAE**: spatio-temporal encoding to a compact latent, with an **implicit neural decoder** reconstructing continuous tactile deformation fields (SIREN/DeepSDF lineage) rather than a fixed grid.

The dataset analysis identifies two properties the authors then design around: **spatial locality** and **contact-driven dynamics**. A t-SNE of the six interaction patterns shows physically interpretable structure — Wiping and Peeling form overlapping clusters (shared shear-dominant mechanics), Assembly forms a well-separated manifold (static normal pressure, localised geometric contact), and Grasping spans a broad region reflecting its intra-class diversity.

## 2. Data curriculum — OmniViTac

| | |
|---|---|
| Trajectories | **21,879** |
| Tasks | **86** |
| Objects | **100+** |
| Interaction patterns | **6**, physics-grounded |

The six patterns are defined by their **force signature**, not their semantics — a genuinely useful taxonomy:

- **Assembly** — static normal pressure, localised geometric contact
- **Cutting** — sustained directed force
- **Adjustment** — torsional and shear forces, slip sensing, in-hand reorientation
- **Peeling** — dynamic continuous coupling of shear and normal force
- **Wiping** — simultaneous normal pressure (adherence) and planar shear (overcoming friction)
- **Grasping** — broad force profiles: precise normal control for fragile items, contact feedback for visually challenging (transparent) objects, normal-shear dynamics for articulated mechanisms

Three collection modes on one cross-embodiment platform (UNFactory 7-DoF xArm): **gravity-compensation kinesthetic** teaching for fine-force tasks (wiping, connector insertion), **GELLO teleoperation** for larger spatial motions, and **TacUMI** — a handheld FastUMI-inspired visuo-tactile device for human demonstrations.

Pipeline discipline: **native sensor frequencies preserved**, precise temporal synchronisation across vision/touch/action, automated trimming, and human-in-the-loop verification.

## 3. Model — a slow-fast hierarchy

| Component | Rate | Role |
|---|---|---|
| Image observation | 15 Hz | input |
| **Slow Policy** (VTWM + fusion) | **4 Hz** | predict contact evolution, generate actions |
| **Reflexive Latent Tactile Controller (RLTC)** | **60 Hz** | correct deviation between predicted and observed tactile |

Four coupled modules: **TactileVAE** → **two-stream conditional generative Visuo-Tactile World Model** (short-horizon contact evolution) → **Adaptive Visuo-Tactile Fusion Policy** → **RLTC**.

## 4. How tactile enters the model

**Latent Tactile Differential (LTD) Encoder** — the distinctive piece. Rather than feeding current tactile or predicted tactile, it encodes the **relationship between current tactile observation and predicted future tactile features**. The policy conditions on the *difference*, i.e. on how contact is about to change.

**Contact-aware gating** then adaptively weights visual against tactile features by the predicted contact probability.

**RLTC** closes the loop at 60 Hz on the residual between predicted and observed tactile latents.

## 5. Experiment setup

Real-robot experiments across all six interaction categories, plus generalisation to unseen objects and geometric configurations, plus deliberate perturbation experiments (abruptly lowering the object to break contact).

## 6. Findings

**LTD beats plain tactile.** Feeding the current-vs-predicted differential achieves higher success than using the current tactile observation alone, across multiple tasks.

**Gating beats concatenation by ~7%** average success. The visualisation is convincing: modality weights track contact state — **tactile weight stays near zero with no contact and rises sharply as predicted contact probability rises**. The policy learns to be visual before contact and tactile during it, which is the behaviour [[feelworld]] hard-codes with a gate and [[vt-wam]] induces with a loss.

**Tactile prediction quality gates everything.** Using world-model checkpoints degraded to 80/60/40/20% of best tactile-prediction performance, success falls monotonically; at **60%** prediction performance the model already fails to estimate future contact probability and mis-weights the modalities. A lighter alternative predicting **only future contact state** instead of the full tactile prediction also causes a significant drop. So the contact-probability scalar is not sufficient — the policy needs the predicted tactile field.

**The negative result — generated visual features don't help.** Adding generated visual features to the policy input yields **no significant gain**, while the visual generation branch **significantly reduces inference frequency**. The final design therefore uses only *current* visual observations plus *predicted* tactile.

This is worth dwelling on. Most of the world-model cluster ([[n0-twam]], [[vt-wam]], [[tactile-wam]], [[dream-tac]]) predicts future vision and future touch jointly and conditions actions on both. OmniVTA measured the visual half and found it not worth its latency. Whether that reflects their scale, their tasks, or a general truth is untested — but it is the only paper here that checked.

**RLTC.** In perturbation experiments the 60 Hz controller re-establishes contact rapidly (recovery-time comparison with and without RLTC), and adaptively modulates action magnitude in high-force tasks to prevent excessive contact force. Removing it drops success.

**Generalisation.** Stable performance under unseen geometric configurations and unseen tools.

## 7. What it adds that the others don't

The **force-signature taxonomy** of interaction patterns (a better organising axis for tactile data than task semantics), the **LTD** representation (condition on predicted *change* in contact, not on contact), and the measured finding that **visual generation is the expendable half of a visuo-tactile world model**. Its cross-embodiment collection across four different tactile sensors also makes OmniViTac one of the few large datasets that is not locked to a single gel.
