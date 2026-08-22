# VQ-Touch — A Data-Efficient Tactile Generation Framework Across Sensors and Scenarios

**arXiv:2607.14728** · Institute of Automation CAS + Zhongguancun Academy (Lyu, L. Xiao, Zeng, D. Wu, Shu, Hao) · Jul 2026

**One line.** Attacks tactile generation from the *data-efficiency* side: model sensors at the **family** level, transfer representations by **few-shot mixed training**, and avoid retraining a generator per sensor.

## 1. The problem

Vision-based tactile sensors have *"limited durability and [a] laborious real-world acquisition pipeline,"* making tactile data scarcer and noisier than visual data. Generation is the scalable surrogate — but two obstacles persist:

1. **Per-sensor pretraining** — existing methods need large diverse datasets *for each sensor type*, so they degrade badly wherever tactile data is hard to acquire.
2. **Non-transferable features** — learned representations do not generalise to generation tasks for other sensors.

## 2. Model — three components

**(a) DM-VQGAN**, a tactile-specific representation learner. The motivation is a real property of tactile images: *"Vision-based tactile images discard color while preserving essential cues such as morphology, geometry, and force distribution"* — low intrinsic variance, which suits a discrete codebook under limited data.

But vanilla VQGAN *"relies on fixed convolutions and thus struggles to capture the non-rigid deformations and multi-scale patterns characteristic of VBTS."* DM-VQGAN adds **deformable convolutions** (for non-rigid gel deformation) and **multi-scale dilated convolutions / fusion** (macro deformation patterns *and* micro texture).

Standard VQGAN training: encoder E, decoder G, patch discriminator D, codebook Z = {z_k}, nearest-neighbour quantisation
```
z_q = q(ẑ) := arg min_{z_k∈Z} ‖ẑ_ij − z_k‖
L_total = ‖x − x̂‖₁ + ‖sg[E(x)] − z_q‖²₂ + β‖sg[z_q] − E(x)‖²₂ + L_per + L_adv
```

**(b) Few-shot mixed training.** The distinctive idea: *"We model tactile sensors at the **sensor-family** level and use clustering and few-shot mixed training to transfer features from one sensor to its family, eliminating full multi-dataset training."* Instead of a shared latent for all sensors ([[htt]], [[tactx]]) or explicit sensor conditioning ([[unitac]]), VQ-Touch groups sensors into families and transfers within them.

**(c) Discrete diffusion decoder** with a **unified conditioning interface** supporting multiple input modalities — tactile images, visual images, and semantic labels — for conditional generation.

## 3. Capabilities

The unified conditioning interface is what makes it multi-scenario:

| Setting | Description |
|---|---|
| **Few-shot generation** | small-scale dataset for a new sensor |
| **Zero-shot transfer** | label-conditioned ("Rock", "Tree") generation on an unseen sensor |
| **Cross-sensor generation** | trained on one sensor, evaluated on a varied sensor |
| **Vision-limited scenarios** | generation where RGB context is degraded |

## 4. Results

VQ-Touch *"surpasses state-of-the-art methods in multiple tasks"* on tactile image reconstruction and generation, **excelling in vision-limited scenarios** — the regime where the RGB-conditioned approaches ([[felt]], [[egotac]]) are weakest by construction, since both depend on visible contact regions.

## 5. What it adds that the others don't

**Sensor-family modelling with few-shot transfer**, which is a different answer to heterogeneity from everything else in this survey: not one universal latent, not per-sensor conditioning, but *clusters of similar sensors sharing representations*. Combined with the discrete-codebook argument — tactile images have low intrinsic variance, so a codebook is the right structure under data scarcity — and the deformable/multi-scale convolutions targeted at non-rigid gel deformation, it is the most explicitly **data-efficiency-oriented** generation method in the survey.

It is also the generation counterpart to [[unitac]]: both synthesise tactile for a target sensor, but UniTac does it by conditioning a large multimodal model on sensor tokens, while VQ-Touch does it by transferring within a sensor family from a handful of examples. Read alongside [[felt]] (RGB→tactile for policy deployment) and [[tacgen]] (latent generation evaluated by representation utility) — three different answers to what generated touch is *for*.
