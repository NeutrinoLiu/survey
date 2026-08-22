# TDP — Tube Diffusion Policy: Reactive Visual-Tactile Policy Learning for Contact-rich Manipulation

**arXiv:2604.23609** · **Meta Reality Labs Research** + Idiap + EPFL + Toronto + Purdue (T. Xue, Rigo, B. Huang, J. Shen, Z. Xu, Colonnese, Memar) · Apr 2026 · [site](https://schortenger.github.io/tube-diffusion-policy/)

**One line.** Diagnoses **action chunking** as the structural obstacle to tactile reactivity and replaces it with a **tube** — a feedback flow around the nominal chunk that permits correction at every timestep.

## 1. The problem

*"Most existing approaches rely on action chunking, which fundamentally limits their ability to react to unforeseen observations during execution. This limitation becomes especially critical in contact-rich scenarios, where physical uncertainty and high-frequency tactile feedback demand rapid, reactive control."*

This is the same diagnosis [[retouch]] makes about stale predictions and [[tacmamba]] makes about frequency mismatch, targeted at the *action* representation rather than the observation one.

## 2. Method

TDP *"bridges diffusion-based imitation learning with **tube-based feedback control**"*, learning a **hybrid action velocity field** that combines diffusion-based generation with **streaming conditional feedback flows**, forming an **action tube** that *"constrains the generated trajectory around the demonstration manifold."*

Two consequences:
- **Reactive control at every timestep**, enabling rapid adaptation to contact uncertainty and disturbance
- **Stepwise correction reduces the required denoising steps**, significantly lowering inference latency — the reactivity and the speed come from the same mechanism

The paper includes a stability argument: the sequence of post-correction errors is shown **uniformly ultimately bounded** (β/(1−α)), which is unusual rigour for a diffusion-policy paper and is what "tube" means formally.

## 3. Architecture

Conditional **1D U-Net**, three resolution levels (256/512/1024 channels), **12 FiLM-conditioned residual blocks**. Observation conditioning via **FiLM** at dimension 3118. **Six ResNet-18 encoders — two visual, four tactile — without parameter sharing.**

| Component | Parameters |
|---|---|
| Policy U-Net | 44.7 M |
| + action heads (excl. encoders) | 105.2 M |
| **Complete model** | **172.8 M** |

Notably, the observation encoders are **67.6M of the 172.8M** — the tactile and visual front-ends are nearly two-thirds the size of the policy, a cost breakdown rarely reported.

## 4. Results

Evaluated on **Push-T** plus three virtual and two physical visual-tactile dexterous manipulation tasks. Consistently outperforms state-of-the-art imitation baselines, with two real-world experiments validating *"robust reactivity under contact uncertainty and external disturbances."*

## 5. What it adds that the others don't

**A control-theoretic fix to the chunking problem.** The rest of the survey works around action chunking with hierarchies ([[touchworld]], [[omnivta]]), residual correctors ([[unitacvla]], [[vitar]]) or intra-chunk re-prediction ([[retouch]]); TDP changes the generative object itself from a trajectory to a *tube around* a trajectory, gets per-timestep reactivity as a property rather than an add-on, and gets lower latency as a side effect. The bounded-error guarantee is a rare formal result in this literature.
