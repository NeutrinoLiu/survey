# HRDexDB — A Paired Human-Robot Dataset for Cross-Embodiment Dexterous Grasping

**arXiv:2604.14944** (v2) · Seoul National University + RLWRLD (Lim, Ha, M. Choi, J. Kim, B. Kim, Jeon, **Joo**) · Apr 2026 · [site](https://snuvclab.github.io/HRDexDB/)

**One line.** Captures **the same 100 objects grasped by a human hand and four different robotic hands**, markerlessly, with 23 synchronised cameras — so cross-embodiment transfer can be studied with the object held constant.

## 1. The gap

*"Existing datasets rarely provide paired, multi-embodiment captures of comparable grasping behavior over shared objects."* Computer-vision datasets cover human hands; robotic datasets cover the robot and *"often leave object motion only partially tracked"*; the few paired human–robot efforts *"do not provide paired captures across multiple dexterous robotic embodiments over shared objects, and often lack markerless RGB observations or tactile signals."*

The embodiment gap is also correctly framed as **two** gaps: human→robot, *and* robot→robot — *"different dexterous hands impose embodiment-specific physical and kinematic constraints, resulting in distinct feasible contact patterns and grasp strategies."*

## 2. Dataset

| | |
|---|---|
| Grasping trials | **2.1 K** |
| Objects | **100**, with scanned high-quality meshes |
| Embodiments | human hand + **4 dexterous robotic hands** |
| Cameras | **21 exocentric** (calibrated) + **2 egocentric** |

Per trial: 3D human hand motion, robot states, **object 6D pose trajectories**, egocentric observations, **contact force signals from tactile-enabled robotic hands**, 3D grasp and affordance annotations, and **success/failure labels**.

The 21-camera rig exists for a specific reason: *"dexterous manipulation induces severe occlusions, making markerless hand reconstruction and object tracking difficult."*

## 3. Benchmarks

**Human-to-robot transfer:**
- **Contact map transfer** — convert human contact patterns into robot-specific contact maps
- **Cross-embodiment grasp retrieval** — a shared latent space retrieving feasible robot grasp priors from human grasps

**Perception under dexterous interaction** — 3D hand pose estimation and object 6D pose estimation under severe hand–object and robot–object occlusion.

**A concrete downstream result:** fine-tuning the **MegaPose** refiner with 100k GSO synthetic samples plus **5.3k HRDexDB robot-grasp annotations**, evaluated on a *held-out* robot-grasp environment from a different system:

| Refiner | ADD (cm) ↓ | ADD-S (cm) ↓ |
|---|---|---|
| Original | 8.23 | 4.40 |
| Fine-tuned | 7.90 | **3.95** |
| Relative | 3.99% | **10.2%** |

So the dataset improves a general-purpose pose estimator on interaction-centric settings — evidence it captures something about occluded manipulation that generic synthetic data does not.

## 4. Stated limitations

- **Tactile heterogeneity** — *"Tactile sensing is available only for robotic hands, and sensor specifications vary across platforms, complicating unified tactile analysis."* Normalisation or shared latent representations are proposed, i.e. exactly the problem [[htt]], [[tactx]] and [[uniforce]] address.
- **Trajectory correspondence** — pairing is at the *semantic* level, and *"defining functionally equivalent motions across different hand morphologies remains an open problem for cross-embodiment imitation."* That is the same objection [[tactalign]] raises against strictly paired human–robot data, stated by a paper that chose to pair anyway.

Expansion toward 1,000 objects and functional manipulation is planned.

## 5. What it adds that the others don't

**Paired multi-embodiment capture over a shared object set**, which is the only design that lets you ask "what changes when the hand changes, holding the object fixed?" Its tactile is a secondary modality rather than the focus, but the resource fills a gap that the human-to-robot tactile-transfer work ([[tactalign]], [[hcttp]]) currently has to work around with unpaired or single-robot data.
