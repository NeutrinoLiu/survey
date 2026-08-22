# EgoScale — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | EgoScale: Scaling Dexterous Manipulation with Diverse Egocentric Human Data |
| **Org** | UT Austin Robot Perception & Learning Lab + collaborators |
| **Date** | 2026-02-18 |
| **Artifact** | arXiv 2602.16710 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper with a fitted, published scaling law** |
| **Corpus** | **20,854 h** action-labeled egocentric human video (>20× prior work) + 829 h EgoDex + 50 h aligned human / 4 h robot |
| **Stance** | **Tolerate noise, buy diversity.** Noisy labels at scale beat clean labels at small scale. |

## 1. The thesis — noise is affordable

EgoScale is the clearest articulation in this survey of the *anti-cleaning* position:

> *"Although these estimates are noisy due to unconstrained data collection, the scale and diversity of the data provide effective supervision for learning transferable action representations, which continue to improve downstream performance as data volume increases."*

There is no rejection cascade. Instead the pipeline invests in (a) an **action representation that is robust to the dominant noise mode**, and (b) a **small clean anchor set** to stabilize the noisy bulk. This is the mirror image of Ψ₀'s strategy.

## 2. Sources & scale

| Stage | Data | Scale | Nature |
|---|---|---|---|
| **I — pretraining** | In-the-wild egocentric mixture | **20,854 h**, 9,869 scenes, 6,015 tasks, 43,237 objects; household/industrial/retail/educational | Large, noisy, unconstrained, **not task-aligned** |
| **I — anchor** | **EgoDex** (Apple Vision Pro) | 829 h, 194 tabletop tasks | High-precision wrist/hand tracking; *"help anchor pretraining while preserving scalability"* |
| **II — mid-training** | Aligned human↔robot play | 344 tasks × ~30 human + ~5 robot traj = **~50 h human / ~4 h robot** | Explicitly embodiment-aligned, matched viewpoints |
| **III — post-training** | Task demos | as few as **1 robot demonstration** | Few-shot |

## 3. Ingestion & label generation

All recordings are egocentric RGB at **30 FPS**. Labels are *derived*, not collected:
- **Off-the-shelf SLAM** → camera pose `T_t^{w←c} ∈ SE(3)`
- **Off-the-shelf hand-pose estimation** → 21 keypoints per hand as rigid transforms `H_t^{c,i} ∈ SE(3)` in camera frame
- Wrist in world frame: `W_t^w = T_t^{w←c} · H_t^{c,1}`

The quality-control burden is therefore shifted onto the *perception pipeline*, and EgoScale accepts its error rate rather than filtering its output.

## 4. Noise-absorbing action representation (the real "cleaning" mechanism)

Two design choices do the work a filter would otherwise do:

**(a) Relative wrist motion, not absolute pose.**
`ΔW_t = (W_0^w)^{-1} · W_t^w`
Differencing within an action chunk **cancels global camera-pose drift** — the dominant SLAM failure mode. Absolute-pose error becomes irrelevant as long as it is locally consistent. The same representation is shared by human demos and robot executions, so it doubles as the cross-embodiment interface.

**(b) Optimization-based retargeting with hard constraints.**
21 human keypoints → **22-DoF Sharpa hand** joint space via an optimization that **enforces joint limits and kinematic constraints**. The constraint set is a *validity filter applied at conversion time*: physically impossible hand configurations cannot be emitted.

This choice is ablated explicitly (Fig. 8): pretraining on **fingertip positions** is inconsistent, because *"small errors in fingertip pose often lead to implausible joint configurations after mapping, causing unstable grasps or contact loss"* on contact-sensitive tasks. **Retargeted joint-space actions are the most consistent representation.** In other words — pick the label space in which upstream noise is least amplified.

## 5. Filtering & QC
No explicit episode-rejection stage is reported. Quality is managed by:
- representation choice (§4),
- the clean **EgoDex anchor** mixed into the noisy bulk,
- **Stage-II aligned data** as a grounding correction rather than a filter.

## 6. Mixture weighting / staging
Three-stage curriculum with progressive freezing:
| Stage | Data | Steps | Batch | LR | Frozen |
|---|---|---|---|---|---|
| I | 20,854 h human | 100K, 256× GB200 | 8,192 | 5e-5 | nothing (fully unfrozen) |
| II | aligned human-robot play | 50K | 2,048 | 3e-5 | VL backbone frozen; only vision encoder + DiT action expert update |
| III | task demos | — | — | — | post-training |

The design *"decouples data scale from embodiment alignment"* — scale in Stage I, alignment in Stage II. That decoupling is what makes the noisy corpus usable without cleaning it.

## 7. Scaling evidence — the headline result

Models pretrained at **1k, 2k, 4k, 10k, 20k hours**, mid-training ablated out to isolate the effect.

**Fitted law:**
```
L = 0.024 − 0.003 · ln(D)          R² = 0.9983
```
where `D` = hours of human pretraining data, `L` = optimal converged validation loss on a held-out set of **2,000 egocentric episodes** (20 timesteps sampled per trajectory, MSE against ground-truth wrist and hand actions).

Supporting observations:
- **1k–2k h models overfit** — loss plateaus then degrades, "indicating overfitting to limited behavioral diversity."
- **10k–20k h models improve monotonically** with no overfitting.
- **Offline loss predicts real-robot performance.** Average task completion rises **0.30 → 0.71 from 1k → 20k hours, monotonically, with no saturation** in the explored regime.
- Transfer holds across embodiments: models supervised in a 22-DoF hand space transfer to the Unitree G1 tri-finger hand.
- Few-shot: **1 robot demo + 100 aligned human demos → up to 88% success on shirt folding**, a task absent from mid-training.

This is the survey's cleanest evidence that a *validation-loss proxy computed on human video* is a usable, cheap predictor of downstream robot performance — which in turn makes data-mixture decisions measurable without running robots.

## 8. What they do not do
- No episode-level rejection, dedup, or VLM-based label auditing.
- No synthetic rendering of robots into human video (contrast Qwen-RobotManip's H2R).
- Perception-pipeline errors are inherited wholesale.
- Scale is reported in hours; no per-source quality accounting is published.

## 9. Transferable takeaways
1. **Choose the representation that cancels your noise.** Relative wrist motion neutralizes SLAM drift for free — cheaper and more robust than detecting and dropping drifted episodes.
2. **Constrained retargeting is a filter.** Enforcing joint limits at conversion time prevents implausible labels from ever existing.
3. **A small high-precision anchor stabilizes a large noisy corpus** (829 h EgoDex inside 20,854 h of wild video).
4. **Held-out action-prediction loss is a cheap, high-fidelity proxy** for downstream policy quality — R² = 0.9983 against data scale, and strongly correlated with real-robot completion.
5. Read against **Ψ₀** (800 h clean > 8,000 h mixed) and **DYNA-2** (1M h). EgoScale sits in the middle and provides the only fitted curve of the three.
