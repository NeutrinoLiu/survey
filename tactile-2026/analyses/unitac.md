# UniTac — A Unified Multimodal Model for Cross-Sensor Tactile Understanding and Generation

**arXiv:2606.31451** · Zhejiang University + Yale + Oxford + MIT + SJTU + UNIX AI (Tu, F. Yang, C. Ma, X. Yu, Zeng, S. Wu, H. Zhao, Tao, C. Zhang, Qian, A. Wong) · Jun 2026

**One line.** The first unified understanding-and-generation model for touch, built on a specific physical insight: **tactile acquisition is a two-stage process** — non-contact (which encodes the *sensor*) then contact (which encodes the *object*) — and a model must represent both levels.

## 1. What "tactile" means here — a dual-level representation

The framing is the contribution:

> *"Unlike visual cameras that capture a natural image in a single exposure, tactile data acquisition inherently involves two stages: a non-contact stage capturing sensor configuration and a contact stage recording object-level physical properties under that configuration."*

- **Sensor level** — lighting, gel deformation baseline, camera parameters, marker layout
- **Object level** — surface geometry, hardness, roughness

And the consequence, stated crisply: *"Without object-level information, non-contact tactile data lack semantic meaning; without sensor-level information, generative models are unaware of the sensor configuration on which tactile signals should be synthesized, while understanding models cannot interpret how sensor design translates tactile patterns into object properties."*

Their example: GelSight encodes surface roughness through **marker displacement patterns that depend on its own sensor configuration** — the same roughness produces different images on different gels.

## 2. Data curriculum

The critique of prior touch-language work is a data-aggregation argument: models are trained on *"small, self-curated datasets, such as PHYSICLEAR with only 482 touch videos"*, while the community has collectively released **over 400K video clips and 1.6 million frames** that are *"typically used in isolation rather than being combined for joint training."*

UniTac trains on the union.

## 3. Model

- **Touch encoder** — AnyTouch-architecture ViT-B/16, pretrained on large-scale multi-sensor tactile video, with **learnable sensor tokens (five per sensor type)** encoding sensor-specific characteristics; outputs 768-d latents carrying both object-level semantics and sensor-level configuration
- **MLLM** — shared backbone for understanding *and* generation
- **Sensor-aware DiT projector** — maps MLLM outputs into the conditional space of the decoder
- **Touch decoder** — synthesises tactile data

**Understanding tasks**, dual-level by design:
1. **Object property description** — physical attributes of the contacted object
2. **Sensor identification** — recognise which sensor configuration produced the signal

**Generation training**, two stages:
1. **Reconstruction** — decoder trained on hidden tokens from the touch encoder, **without involving the MLLM**, enabling efficient parallel training
2. **Alignment** — DiT projector aligns MLLM output with encoder representations; **sensor-level tokens from the encoder are injected into the projector** to compensate for the absence of sensor cues in natural-language tactile descriptions

**Sensor-prior-based sampling** simulates the non-contact→contact transition at inference, conditioning generation on both levels.

## 4. Results

**Understanding and generation.** UniTac-7B is strongest overall on the six PHYSICLEAR-Test understanding tasks and leads on generation SSIM/PSNR.

**Ablation asymmetry, cleanly reported:** removing components trained *jointly with the MLLM* hurts both understanding and generation — *"tactile comprehension not only improves reasoning performance but also provides critical semantic priors for generation."* Removing the DiT projector or sensor-prior sampling (trained independently of the MLLM) barely touches understanding but *"reduces generative fidelity"* — sensor-level representation is what maintains structural realism and physical consistency.

**Real-world deployment — language-guided fabric selection.** The task: pick the fabric most suitable for wiping an infant's skin from two **visually similar** options with different tactile properties, then grasp a single-layer edge, lift stably, and avoid slipping.

| Method | Selection ↑ | Grasping ↑ | **Overall ↑** |
|---|---|---|---|
| VLA (RGB only) | 11/20 (55%) | 6/20 (30%) | **4/20 (20%)** |
| VTLA-real (real GelSight at rollout) | 20/20 (100%) | 19/20 (95%) | **19/20 (95%)** |
| VTLA-pred (tactile predicted from RGB/state) | 18/20 (90%) | 16/20 (80%) | **16/20 (80%)** |

The RGB baseline localises fabric regions but fails at both **tactile-semantic selection** and **single-layer contact-state estimation** — 20% end to end. And **VTLA-pred at 80% without any real-time tactile input** is another instance of the pattern seen in [[hapticvla]], [[unitacvla]] and [[at-vla]]: predicted tactile representations retain most of the benefit.

**Generated data closes a sensor gap — the most practical result:**

| Training data | Digit grasp acc. | **GelSight grasp acc.** |
|---|---|---|
| Digit only | 98.89% | **50.00%** (chance) |
| Digit + UniTac-generated GelSight | 99.07% | **99.37%** |

A classifier trained on Digit is **at chance on GelSight**, quantifying the sensor gap in the same way [[tacverse]] does with negative R². Augmenting with **synthetic** GelSight data recovers 99.37% without collecting any real GelSight data, and Digit performance is unharmed. Given how fast tactile hardware turns over, generating adaptation data rather than recollecting it is a genuinely useful capability.

## 5. What it adds that the others don't

The **sensor/object decomposition** grounded in the physics of tactile acquisition, and the demonstration that **generation solves a problem understanding cannot** — cross-sensor adaptation for a newly released device with zero real data on it. Where [[htt]] and [[ftp-1]] make representations sensor-agnostic, UniTac makes the sensor an explicit, controllable conditioning variable, so it can *render into* a target sensor. Its 50% → 99.37% augmentation result is the strongest evidence in this survey that synthetic tactile data is usable, not just plausible.
