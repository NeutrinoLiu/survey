# T-Rex — Tactile-Reactive Dexterous Manipulation

**arXiv:2606.17055** (v2) · UC Berkeley + **NVIDIA** + Stanford + Panasonic + La Sapienza + ItalAI (Niu, Z. Liu, Z. Wang, … Fei-Fei, Goldberg, Malik, Abbeel, Y. Zhu, D. Xu, Fan, Darrell) · Jun 2026 · [site](https://tactile-rex.github.io/)

**One line.** Argues tactile capability can be acquired in a **dedicated mid-training phase** rather than during pretraining — 22,889 hours of human video for semantics, then 100 hours of tactile robot data to ground it.

## 1. The framing — *tactile-reactive*

*"Mastering [these tasks] requires **tactile-reactive behaviors**: immediate, closed-loop motor responses to tactile signals, far faster than conventional vision-based control loops allow."*

Two obstacles: tactile pretraining data is scarce, and *"learning such policies from scratch would require massive amounts of synchronized visuo-tactile data — a scale at which collecting fine-grained dexterous teleoperation becomes prohibitively expensive."*

The resolution is the paper's structural claim: **tactile does not need to be in pretraining**. Human video supplies semantics and coarse visuomotor priors; a mid-training phase on tactile robot data bridges those priors to contact-rich control.

## 2. Data — a deliberate collection philosophy

| Source | Scale |
|---|---|
| Human egocentric video | **22,889 hours** |
| T-Rex Dataset (teleoperation) | **100 hours**, 7,700+ trajectories |
| Motor primitives | **200+** |
| Tasks | 12 |

The collection recipe is the notable part: *"Rather than recording narrow, task-specific demonstrations, we design it around **diverse verb–noun combinations**, covering contact-rich behaviors through compositional motor primitives and object interactions."* Skills span close, peel, wrap, fold, wipe, squeeze, insert, extract.

## 3. Model — variable-rate MoT

A **Mixture-of-Transformer-Experts** backbone with three experts running at different rates:

| Expert | Function | Rate |
|---|---|---|
| **Latent expert** | future visual latent prediction | — |
| **Action expert** | slow action denoising | **~5 Hz** |
| **Tactile expert** | fast tactile refinement | **~20 Hz** |

Plus a **spatial-temporal tactile VQ-VAE encoder** — a *temporal* discrete encoder, contrasting with the *static* encoders the paper identifies as a limitation of prior work.

The variable-rate design is the mechanism for exploiting *"naturally high-frequency touch signals without sacrificing the existing capabilities of existing VLAs"* — the same problem [[tacmamba]] solves with a state-space model and [[touchworld]] with a control hierarchy.

## 4. Results

**+30% average success** over the strongest baseline across 12 real tasks requiring delicate force control and deformable-object manipulation, with significantly better data efficiency and zero-shot capability.

**Stage ablation:** *"human pretraining provides broad semantic grounding and coarse visuomotor priors, while tactile-grounded mid-training bridges these priors to robot-executable contact-rich control."* Both stages contribute; the full recipe is best.

## 5. Stated limitations — including a candid hardware verdict

- Long-horizon tasks with tight tolerances where teleoperation is hard would benefit from RL or online refinement.
- *"Tactile-reactive manipulation remains **bottlenecked by hardware**, including sensor distortion, calibration drift across devices, and the **absence of dense palm sensing** for whole-hand manipulation."*

That last point is a direct corroboration, from a large real-robot effort, of [[tactilegenesis]]'s simulation finding that palm and proximal-phalange coverage matters more than fingertip resolution.

## 6. What it adds that the others don't

The **mid-training** position: touch need not be in the foundation-model pretraining corpus, only in a bridging stage — which changes the economics substantially, since 100 hours of tactile teleoperation is achievable where 22,889 hours is not. The **variable-rate MoT** with a temporal VQ-VAE tactile encoder is also the cleanest architectural answer here to the frequency mismatch, and it is validated at a scale and author density that make the result hard to dismiss.
