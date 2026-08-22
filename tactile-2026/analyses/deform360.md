# Deform360 — A Massive Multi-view Visuotactile Dataset for Deformable World Models

**arXiv:2607.05390** · Brown · Columbia · MIT (H. Li, Fu, Cong, Z. Li, B. Huang, Jiang, He, Liang, R. Fu, Lu, Sridhar, Smith, Konidaris, Y. Li) · Jul 2026 · [site](https://deform360.lhy.xyz)

**One line.** 41 cameras and two tactile grippers around 198 deformable objects, built to settle whether 2D video world models or 3D particle models are the right substrate — and the answer is *it depends entirely on how much data you have*.

## 1. What "tactile" means here

Tactile signals `T ∈ R^{T×D}` from sensors on **bimanual tactile-equipped UMI grippers**, software-synchronised to the visual stream at **30 Hz**. The sensors measure **normal-axis pressure**.

Tactile plays an unusual role: it is not a policy input but an **annotation-refinement signal**. The tracking pipeline uses tactile to enforce a **no-slip regulariser** on particles near contact. The authors are honest about the consequence — because the sensors only read the normal axis, they *cannot identify micro-slip*, so when visible slip actually occurs the tactile regulariser **over-constrains nearby particles**. That is a rare, specific admission of how a tactile prior can corrupt ground truth.

Visual side: **N = 41 surround-view cameras**, 720×1280 at 30 fps, each with calibrated intrinsics `K_n ∈ R^{3×3}` and extrinsics `E_n ∈ R^{4×4}`. The density exists specifically to resolve contact-induced local deformation that end-effectors and self-occlusion hide.

## 2. Data curriculum

| | |
|---|---|
| Objects | **198** daily-life |
| Sequences | **1,980** (5 unimanual + 5 bimanual per object) |
| Frames | **23.3 M** across all views (614,490 per viewpoint avg.) |
| Raw videos | 74,850 |
| Duration | **215.7 h** cumulative multi-view; 20,483 s single-view; 10.34 s avg. episode |

Object taxonomy by material response:
- **1D deformables** — 28 ropes, cables, wires, varying stiffness/thickness
- **2D deformables** — 98 fabrics, garments, bags, paper-like thin shells
- **3D volumetric** — 72 plush toys, foam, squeezable objects

Manipulation primitives: unimanual poking and squeezing; bimanual stretching, folding, twisting. Robot proprioception (6D pose + openness) recorded alongside.

**Annotation pipeline** — the technical contribution. Per-frame **3D Gaussian Splatting** recovers high-fidelity geometry; markerless 2D tracking per view is then **lifted into 3D using that geometry** and optimised for multi-view/temporal consistency and physical plausibility, with tactile supplying the contact-region constraint. Geometry recovery and temporal tracking are deliberately decoupled because GS is optimised for rendering, not tracking, and independent Gaussian motion yields temporally inconsistent trajectories.

Against prior datasets, Deform360 is the only one with **all** of mesh, calibration, markerless tracking, tactile, and 360° coverage; nearest competitors Robo360 (~2M frames, no tactile, no high-fidelity annotation) and PokeFlex (21.3k frames, tactile, limited actions).

## 3. Models benchmarked

Not a model paper. It evaluates:
- **3D particle** — PGND (MLP-based), ParticleFormer (transformer), PhysTwin (physics-based optimisation)
- **2D video** — Cosmos (action-conditioned video diffusion)

## 4. How tactile enters

Only through annotation, plus a dedicated **contact-prediction** task that models the coupling between visual observation and physical contact events. Neither the video nor the particle world models consume tactile at inference.

## 5. Experiment setup

Three generalisation regimes, deliberately laddered:

1. **Single-episode** (low data) — reconstruction/resimulation + future prediction on held-out frames.
2. **Multi-episode** (episode generalisation) — train on E_train episodes of an object, test on unseen episodes of the same object.
3. **Multi-object** (zero-shot object generalisation) — train on O_train objects, test on entirely unseen ones.

Plus **real-world MPC planning** deployed zero-shot on a *different robot (xArm) in a different lab*.

Metrics: Chamfer distance, track error, PSNR, SSIM, LPIPS.

## 6. Findings

**Low data → physics priors win.** Single-episode:

| Method | CD ↓ | Track err ↓ | Pred CD ↓ | Pred track err ↓ |
|---|---|---|---|---|
| PGND | 0.032 | 0.033 | 0.073 | 0.073 |
| ParticleFormer | 0.039 | 0.034 | 0.044 | 0.041 |
| **PhysTwin** | **0.014** | **0.021** | **0.014** | **0.025** |

PhysTwin roughly halves the error. Cosmos is excluded outright — per-episode data is insufficient for post-training to produce stable predictions.

**Episode generalisation → the split appears.** Cosmos wins *reconstruction* (PSNR 27.748 vs 26.263) because training in pixel space preserves texture that particle-to-image rendering loses; ParticleFormer wins *future prediction* (CD 0.051 vs — , track err 0.079). Visual synthesis and physical dynamics come apart.

**Zero-shot object generalisation → scale wins.** Cosmos leads on image quality (PSNR 25.042 vs 23.312 ParticleFormer, 22.049 PGND) — large-scale pre-training lets it infer dynamics for unseen categories. PGND collapses (CD 0.429, err 0.320). PhysTwin cannot participate at all: its optimisation fits physical parameters to a specific geometry.

The most interesting failure mode is Cosmos's: it fails to **strictly follow robot commands** over long horizons, yet the dynamics it produces *remain physically reasonable*. It is an action-alignment problem, not a physics problem.

**Planning reverses the ranking again.** Only PhysTwin is deployed for real-world MPC. Cosmos is not, for two stated reasons: video models are sensitive to cross-environment appearance shift at the authors' post-training scale, and **designing a reward on generated video is non-trivial** where 3D models can use Chamfer distance directly. Structural priors that lose the scaling contest win the deployment one.

**Stated limitations.** Heavy self-occlusion still degrades tracking; highly plastic materials violate the local rigidity/smoothness assumptions in particle optimisation; and the normal-only tactile cannot see micro-slip.

## 7. What it adds that the others don't

The only dataset large and dense enough to run the **2D-vs-3D world-model comparison across three data regimes** on real deformables, and the only one that reports the full non-monotonic answer instead of picking a winner. Its use of tactile as a *tracking constraint* — with an honest account of when that constraint is wrong — is a distinct third role for touch alongside observation ([[contactworld]]) and reward ([[tactidex]]).
