# TaF-VLA — Tactile-Force Alignment in Vision-Language-Action Models for Force-aware Manipulation

**arXiv:2601.20321** (v2) · Beihang + ShanghaiTech + BIGAI + HKU (Y. Huang, Lin, W. Li, D. Li, J. Li, J. Jiang, Xiao, Jiao) · Jan 2026 · [site](https://peilin-666.github.io/projects/TaF-VLA/)

**One line.** Proposes a **paradigm shift from tactile–vision alignment to tactile–force alignment**: ground tactile representations in the *physical quantity* they measure, not in the visual scene — backed by an automated rig producing **10M synchronised tactile-force pairs**.

## 1. What "tactile" means here

**Vision-based tactile sensors (VBTS) aligned to 6-axis F/T and matrix force maps.**

The argument proceeds by elimination. **Global force sensors** (wrist F/T, joint torque) have two flaws: high-precision F/T is *"expensive, fragile, and difficult to deploy at the scale required for great data-collection efforts"*, and more fundamentally the data is **low-dimensional and spatially aggregated** — a single resultant vector *"compresses complex contact interactions into a few scalar values, discarding the spatially distributed pressure patterns and local contact geometry."*

**VBTS** supply the missing local dynamics — geometry, texture, local pressure distribution, slippage — at low cost, and their visual nature makes them compatible with vision-based learning.

But then the standard mistake: implanting tactile streams as raw visual tokens aligned with language and scene images *"overlooks the unique modality of tactile sensing: unlike scene vision, which captures remote photometric properties, tactile sensing captures local mechanical interactions."*

Hence the shift: **align tactile to force, not to vision.**

## 2. Three problems the paper poses, and answers

- **Q1 Data scarcity** — no existing dataset has synchronised visuotactile + ground-truth force. How to get it at scale?
- **Q2 Representation disparity** — visuotactile is high-dimensional and geometric; force is low-dimensional and dynamic. **Explicit force regression** (risking poor cross-sensor generalisation) or **implicit latent alignment** (requiring a representation that captures history-dependent dynamics while resisting sensor noise and hardware variance)?
- **Q3 Policy integration** — how to fuse without causing **modality collapse or diluting the physical signal**?

## 3. Data curriculum — the TaF-Device and TaF-Dataset

The acquisition rig is the enabling contribution. A **parallel actuation structure applies forces of the same magnitude and direction simultaneously to both the tactile sensor and the force sensors**, so pairs are physically synchronised rather than temporally aligned after the fact. **Replaceable indenters** varying in stiffness and geometry enrich contact diversity. A modular design allows swapping VBTSs — data is collected on **six distinct sensors**.

Throughput: **100,000 synchronised tactile-force frames per hour**, yielding **TaF-Dataset: >10 million** synchronised (visuotactile image, 6-axis force/torque, matrix force map) triples.

For context, that is roughly two orders of magnitude more force-paired tactile data than any calibration dataset in the prior literature, obtained by automating away the human.

## 4. Model — the TaF-Adapter

A tactile sensor encoder that **extracts discretised latent information** (VQ-style, in the lineage of VQ-VAE / vq-wav2vec / VQ time-series models the paper cites) for encoding tactile observations, pretrained to align sequential tactile observations with ground-truth force in a shared latent space.

The answer to Q2 is therefore **implicit latent alignment with a discrete bottleneck**, not explicit force regression. The stated purpose of the discretisation: *"ensures that the learned representations capture history-dependent, noise-insensitive physical dynamics rather than static visual textures."*

Two properties follow from choosing latent alignment over regression:
- **Cross-sensor robustness** — regression to absolute force values is brittle across gels; a latent aligned to force is not.
- **Noise insensitivity** — the discrete codebook quantises away hardware variance.

The force-aligned encoder is then fused into a VLA backbone and fine-tuned on real demonstrations **enriched with force-aware language instructions**.

## 5. Experiment setup

Real-world contact-rich tasks including **tool-use and deformable object manipulation**, against **tactile-vision-aligned** baselines (the direct comparison the paradigm claim requires) and vision-only baselines.

## 6. Does it work?

TaF-VLA *"significantly outperforms state-of-the-art tactile-vision-aligned and vision-only baselines on contact-rich tasks."* The important comparison is the first one: beating a tactile-vision-aligned model with the same sensor and the same backbone isolates the **alignment target** as the variable.

## 7. What it adds that the others don't

Two things. **(a)** The **alignment-target argument**, stated cleanly and tested: contrastive visuo-tactile alignment (UniTouch, AnyTouch, Touch100k) maximises what touch shares with vision, which is precisely the part vision already has — aligning to force instead targets what vision *lacks*. [[restacvla]] reaches the same conclusion from predictive coding; TaF-VLA reaches it from measurement. **(b)** The **automated tactile-force acquisition device** at 100k frames/hour across six sensors, which is the practical reason the 10M-scale alignment is possible at all. That rig, more than the model, is what makes the paradigm testable — and it is the same physical-units strategy [[n0-twam]]'s NeoForce uses to solve cross-sensor transfer.
