# ContactWorld — What Matters in Vision-Tactile World Models for Contact-Rich Manipulation

**arXiv:2606.13877** · Purdue University + Texas A&M (Z. Zhang, P. Zhou, K. Zhang, Desai, Amosa, Soleymanzadeh, Lei, M. Zheng, She) · Jun 2026 · [site](https://contact-world.github.io)

**One line.** The controlled study the world-model cluster needed: hold architecture and planner fixed, vary the *representation*, and find that **tactile helps only when its dynamics are compatible with the paired visual representation** — and that it helps more the longer the horizon.

## 1. What "tactile" means here — three representations, deliberately

From one vision-based tactile sensor, three derived representations that differ in structure:

| Representation | What it captures |
|---|---|
| **TacRGB** | local tactile appearance |
| **TacDepth** | local surface deformation |
| **TacFF** (force field) | spatial force responses — shear and normal patterns over the tactile surface |

Paired against three visual representations: **wrist view**, **front view**, **point cloud**.

The paper introduces a three-property taxonomy that is the most useful conceptual contribution here:

| Modality | Spatiality | Continuity | Contact sensitivity |
|---|---|---|---|
| Wrist view | medium | **low** (occlusion breaks it) | low |
| Front view | medium | high | low |
| **Point cloud** | **high** | **high** | low |
| TacRGB | low | medium | medium |
| TacDepth | medium | medium | medium |
| **TacFF** | medium | medium | **high** |

The USB-insertion walkthrough makes this concrete: as the robot moves approach → align → insert, the **wrist camera loses the socket to occlusion**, breaking task-relevant visual continuity, while front view retains global visibility and point cloud additionally supplies explicit alignment geometry. On the tactile side, TacDepth and TacRGB change *smoothly* across phases (limited variation), while **TacFF shows clear phase-dependent force responses** during alignment and insertion.

## 2. Data curriculum

**12 contact-rich tasks** across four families:

- **Insertion** — precise geometric alignment, sustained contact, narrow tolerances under partial observability
- **Disassembly** — object separation under frictional/constrained contact, directional resistance, interaction asymmetry
- **Screwing** — continuous rotational interaction, long-horizon consistency
- **Exploratory interaction** — active perception, inferring object properties tactilely

Synchronised visual and tactile observations across all modality combinations.

## 3. Model

A JEPA-style **latent world model** — latent fusion of multimodal observations, latent rollout, with a training objective combining prediction loss, **IDM loss** (inverse dynamics), and regularisation (VC loss, similarity loss). Planning is **CEM** (M candidates, top-K, N iterations) in receding-horizon MPC.

Model size varies with modality: 1.6M – 17.9M parameters. Notably, planning frequency is reported alongside success — a modality that wins on success but is too slow to plan with is not a win.

## 4. How tactile enters the model

By latent fusion, identically across all combinations — that uniformity is what makes the comparison valid. The variable is *which representation* is fused, not how.

## 5. Experiment setup

Full cross-product of {wrist, front, point cloud} × {none, TacDepth, TacRGB, TacFF}, evaluated at goal offsets of **12, 24, 36, 48 steps**, 100 trials per configuration across all 12 tasks. Plus autoregressive latent rollout prediction MSE as a function of horizon.

## 6. Findings

**(a) Spatial structure + temporal continuity wins.** Average planning success: wrist view **20.7%**, front view **22.0%**, point cloud **32.1%**.

**(b) Tactile is not universally beneficial — compatibility decides.** PointCloud + TacFF reaches **36.1%**, the best overall. But the task-level breakdown is where the real finding is:

- **Image-based tactile representations can *reduce* performance on insertion tasks** — local appearance cues provide less stable interaction structure during contact alignment.
- **TacFF** gives the most consistent gains on **insertion and disassembly**, where evolving contact dynamics and friction dominate.
- **TacDepth** performs strongly on **screwing and exploration**, where local geometric deformation and contact-state awareness matter more than force evolution.
- **TacRGB** is variable everywhere — appearance-based tactile is the least stable predictive structure.

So the right tactile representation is task-dependent, and the wrong one is worse than none.

**(c) Long horizons expose representation quality.** Success by goal-offset step (avg over 12 tasks, 100 trials):

| Offset | Wrist only | Wrist+FF | Front only | Front+FF | PC only | **PC+TacFF** |
|---|---|---|---|---|---|---|
| 12 | 41.4 | 44.7 | 44.8 | 45.3 | 52.1 | **54.4** |
| 24 | 21.1 | 26.2 | 23.7 | 23.8 | 36.6 | **41.6** |
| 36 | 12.3 | 14.3 | 11.7 | 12.9 | 23.7 | **27.8** |
| 48 | 8.2 | 8.0 | 7.9 | 7.8 | 16.0 | **20.5** |

Everything collapses with horizon — wrist-only 41.4% → 8.2%, front-only 44.8% → 7.9%. Point cloud degrades most gracefully. And the **tactile gain grows**: PC+TacFF is +2.3 points at 12-step offset but **+4.5 points at 48-step**, a relative gain rising from 4% to 28%.

Note the counterpoint: at 48 steps, **wrist+FF (8.0) and front+FF (7.8) are both *worse* than their vision-only baselines**. Tactile only compounds favourably on top of a spatially structured visual representation.

**(d) The mechanism.** Autoregressive rollout MSE on USB insertion increases monotonically with horizon for every combination, but **adding tactile reduces error and slows accumulation**. That is the predictive explanation for (c): touch improves temporal consistency of latent rollout, which matters more the further ahead you plan.

**Stated limitations.** Tasks are largely single-stage and goal-directed rather than hierarchical/multi-stage; and even 48 steps is short relative to practical manipulation, with performance already at 20.5% at the best configuration.

## 7. What it adds that the others don't

The **cross-product**. Every other paper here proposes one fusion design and shows it beats a baseline; ContactWorld varies representation while holding architecture, planner and tasks fixed, and produces the field's most transferable design rule: *pair force fields with point clouds, and don't expect tactile RGB to help a wrist camera.* Its horizon sweep is also the empirical basis for the claim other papers assert — that tactile matters most where errors compound. Compare [[disentvtf]], which isolates the interface rather than the representation.
