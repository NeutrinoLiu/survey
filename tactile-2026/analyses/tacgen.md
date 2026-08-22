# TacGen — Touch Is a Necessary Dimension of Physical-World Representation

**arXiv:2606.29173** (v2) · University of Maryland + CMU + Alabama + UW + Stanford (Ye, Das, S. Chen, … A. Li) · Jun 2026

**One line.** The most statistically rigorous paper in this survey. It asks whether touch is *necessary* rather than merely helpful, and answers with matched backbones, five-seed reproductions, permutation controls, and a capacity sweep proving vision-only scaling cannot close the gap.

## 1. The question

*"Two visually identical objects — a sponge and a brick rendered at the same scale — share appearance but not physics. Scaling a vision-only representation does not by itself recover information the camera did not observe."*

Grounded in psychophysics (Klatzky & Lederman's exploratory procedures: lateral motion for texture, pressure for compliance, contour following for shape, enclosure for volume) and neuroscience (visuo-haptic convergence in lateral occipital complex). The engineering question is whether representation learning reproduces that grounding.

## 2. Method

Two coupled components on paired RGB + **DIGIT** tactile from the SSVTP/TVL corpus:

1. **V+T contrastive alignment** with a **frozen DINOv2** backbone, samples paired by id under SHA-256-verified preprocessing, with **corrected tactile background subtraction as part of the model definition**.
2. **V→T generator** — a **latent-space residual-MLP diffusion** model synthesising tactile DINOv2 features from RGB tokens, for tactile-data scaling.

The comparison is deliberately conservative: same split, same frozen backbone family, same lightweight probe protocol for vision-only and V+T.

## 3. Results

**Physical-property probes** — V+T over matched V-only, same feature budget, bootstrap CIs excluding zero:

| Property | Improvement |
|---|---|
| Controlled **mass** | ΔR² = **+0.570** |
| Controlled **density** | Δacc = +0.067 |
| SSVTP **hardness** | Δacc = **+0.117** |
| Uncertainty-banded **force** | ΔR² = **+0.281** |

Five-seed reproductions: force **+0.292 ± 0.011** (bracketing +0.281), hardness **+0.102 ± 0.010**, **5/5 positive**. Direction persists across YCB-Sight transfer, palpation grip-force, and **three backbone families** (DINOv2, CLIP+MAE, Sparsh).

**The manipulation test, and its control** — TACTO behaviour cloning:

| Condition | Success |
|---|---|
| V-only (matched capacity, H=128, E=200) | 0.2456 |
| V-only at **8× width / 2× epochs** (24-cell sweep) | 0.279 ± 0.004 |
| **V+T** | **0.979** |

Vision-only capacity scaling accounts for **only 4.5% of the gap, preserving 95.5%**. Since demonstrations, hidden width and budget are matched and *"the only added input is the tactile DINOv2 token"*, the ~4× jump measures the contribution of contact-state information directly. This is the cleanest refutation in the survey of the "just scale vision" position.

Independently corroborated by palpation grip-force regression on 958 episodes: paired ΔR² = +0.169, **5/5 seeds positive**.

**Generation, evaluated by utility not realism** — the methodological point worth stealing. Generated tactile latents reach cross-seed **ΔR²_gen = +0.589**, with the protocol-matched **real** tactile point at **+0.585 inside the seed interval**: synthetic tactile latents are as useful as real ones for these probes.

And the comparison that justifies the latent-space design: a **pixel-space U-Net DDPM** produces tactile-*looking* output but shows a **13 percentage-point downstream utility gap**. Reconstruction quality and representation utility are different things — the same lesson [[tactile-wam]] reaches via pixel–contact misalignment and [[vitacworld]] via the tactile-PSNR caveat, here measured as a direct utility gap.

## 4. The control suite

This is what separates the paper from the rest of the cluster:

| Setting | Controls |
|---|---|
| Probes | tactile permutation, label permutation, random features — separating contact evidence from dimensionality or label artifacts |
| Generation | matched-pair and shuffled tests — separating useful latents from plausible texture |
| Action | V-only capacity scaling, three frozen backbones, 2-layer probes — bounding the vision-only alternative |
| Labels | cross-corpus measured-force validation, **R² = 0.67 on Sparsh GelSight, 35,000 ATI Nano17 samples** |
| Reproducibility | SHA-256-verified manifests, canonical loaders, five-seed reproductions |

Because calibrated per-sample F/T labels are not standardised across public visuo-tactile corpora, they release a reusable **uncertainty-banded label framework** with explicit **p05/p50/p95 intervals and per-sample provenance** — a genuinely reusable artifact for a field where force labels are the scarce resource.

They even report compute carbon (~25.5 kgCO₂eq).

## 5. Stated limitations

DIGIT-style object-contact imagery and frozen encoder families only, not every tactile geometry; force analysis uses uncertainty-banded rather than calibrated labels; TACTO is a controlled test and broad real-robot deployment across hands, sensors and task families remains future work.

## 6. What it adds that the others don't

**Evidential rigour.** Almost every paper in this survey reports "tactile helps" from a single-seed comparison against one baseline. TacGen asks whether the effect survives permutation controls, seed variance, backbone substitution, and — decisively — **vision-only capacity scaling**, and shows it does. Its practical prescription is worth adopting field-wide: *"report V-only baselines, but treat aligned touch as the reference point when the target property is defined by contact."* The generated-vs-real latent equivalence (+0.589 vs +0.585) is also the strongest evidence here that synthetic tactile can substitute for measured tactile at the representation level — compare [[unitac]]'s 50% → 99.37% cross-sensor augmentation result.
