# VTAM — Video-Tactile-Action Models for Complex Physical Interaction Beyond VLAs

**arXiv:2603.23481** · UIUC + Stanford + SJTU (H. Yuan, Yi, Z. Zhang, W. Chen, Mo, Yin, X. Li, Zeng, Wen, Lu, Driggs-Campbell, Lourentzou) · Mar 2026 · [site](https://plan-lab.github.io/vtam)

**One line.** Drops language entirely — a **Video**-Tactile-Action model rather than a VLA — on the argument that a semantic embedding space optimised for visual alignment is the wrong place to learn contact physics.

## 1. What "tactile" means here

High-resolution tactile treated as a **primary sensory modality**, jointly predicted alongside vision in a shared latent space, conditioned on end-effector state. Notably: **no additional wrist force sensor needed**, and **zero tactile pretraining on real data**.

The critique of prior integration is the most precisely argued in this survey. Existing tactile-VLAs either project tactile embeddings into a pretrained vision-language latent space as semantic tokens, or concatenate tactile features with language-conditioned visual representations downstream. Both *"place a substantial burden on representation learning: the model must implicitly infer contact physics within a semantic embedding space optimized for visual alignment and static scene description rather than physical prediction."*

Two consequences follow, and both are correct:

- Learning that a tactile pattern means *slip* must be discovered indirectly through static correlations — demanding large annotated datasets with no guarantee the high-frequency dynamics survive.
- **Without explicit temporal modelling, representations cannot encode causal relationships between successive tactile frames** — precisely the structure needed to anticipate incipient slip.

Hence: predict the future of both streams instead, which forces temporal consistency without semantic annotation of contact events.

## 2. Data curriculum

Strikingly small: **10 minutes of teleoperation per task**, three contact-rich tasks — **potato chip pick-and-place** (fragile), **cucumber peeling**, **whiteboard wiping** with varying heights and tilt angles.

The entire method is a **lightweight modality-transfer finetuning** of a pretrained video transformer, *"without tactile–language paired data or independent tactile pretraining."* That is the practical claim: video-derived dynamics priors substitute for tactile pretraining.

## 3. Model

**Vision-tactile world model** on a pretrained video backbone → predicts future vision, future tactile, future states, future forces. **Action model** = diffusion-based control policy consuming those predictions plus past action and state.

Two mechanisms:

1. **Joint visuo-tactile prediction** in a shared latent space, conditioned on end-effector state — temporally consistent contact dynamics without semantic labels.
2. **Virtual force prediction objective at the action head** — a tactile regularisation loss enforcing balanced cross-modal attention, explicitly to prevent **visual latent dominance**.

The modality-collapse framing is grounded in the multimodal-learning literature the paper cites (Wang et al. on why multimodal classification training is hard; Wu et al. on the greedy nature of multimodal learning) — this is a known pathology being imported into robotics, not a robotics-specific discovery.

## 4. Results

**Average success 90%** across the three tasks. The chip pick-and-place ablation is dramatic:

| Method | Chip pick-and-place |
|---|---|
| Vision-only baseline (π₀.₅) | **0%** |
| Naive downstream force integration (no predictive visuo-tactile modelling) | **0%** |
| VTAM w/o virtual force regularisation | **10%** |
| **VTAM** | **90%** |

Read the middle two rows. **Naive force integration achieves exactly what vision-only achieves: nothing.** And predictive visuo-tactile modelling *without* the regularisation gets only to 10% — so both components are necessary and neither is sufficient. The paper reports +80% over π₀.₅.

Similar trends on peeling and wiping.

A potato chip is a well-chosen task: the margin between "gripping firmly enough to lift" and "crushing" is narrow, invisible to a camera, and unforgiving — a task where partial credit does not exist.

## 5. What it adds that the others don't

**Dropping language.** Every other model in this VLA cluster inherits a vision-language backbone and then fights the semantic prior it brings; VTAM starts from a **video** backbone whose pretraining objective is already *prediction*, and argues that this is the right prior for contact physics. The 0% / 0% / 10% / 90% ladder is the cleanest demonstration in this survey that tactile access, predictive modelling, and anti-collapse regularisation are three separate requirements — missing any one leaves the policy at or near chance.

Its 10-minutes-per-task data budget also makes it the most sample-efficient result here, which is the strongest evidence for the claim that video-derived dynamics priors transfer to touch. Compare [[restacvla]] and [[vt-wam]], which diagnose the same modality-collapse pathology and fix it by decorrelation and by architecture respectively.
