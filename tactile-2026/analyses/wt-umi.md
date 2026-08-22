# WT-UMI — Tactile-based Whole-Body Manipulation via Force-Supervised Contact-Aware Planning

**arXiv:2606.13232** · Institute for Robotics and Intelligent Machines, **Georgia Tech** (Jang, Z. Gu, Cueva, Chai, Sheng, Nguyen, … Y. Zhao) · Jun 2026 · [site](https://wt-umi.github.io/WTUMI/)

**One line.** Makes contact force a **planned, predicted and regulated** quantity for whole-body humanoid manipulation, and resolves the human-vs-teleoperation demonstration trade-off with one wearable interface that serves both.

## 1. The setting

Tasks like *"carrying a large box, reorienting a soft pillow, or transporting a beam with a human partner cannot rely on grasping alone; instead, they require coordinated contact across the torso, forearms, and hands to distribute interaction forces."* And *"small errors in contact location or force allocation can lead to slips, collisions, or load loss."*

Sensing gap: *"vision is often occluded during contact and does not directly measure interaction forces, while **proprioception cannot localize body-surface contact**."*

## 2. The demonstration trade-off it resolves

| Source | Captures | Missing |
|---|---|---|
| **Human demonstration** | natural contact forces | robot-executable actions |
| **Teleoperation** | robot actions | natural force regulation |

**WT-UMI** is a wearable whole-body tactile interface **worn by a human operator or mounted on the humanoid**, giving consistent observations of tactile images, contact forces and end-effector poses across *both* modes. One hardware, two data regimes — the same shared-embodiment principle as [[realdexumi]] and [[twins]], applied to the torso.

## 3. Method

- **Force-conditioned target-pose correction** — converts measured *human* poses into contact-aware *robot* targets by learning corrections from teleoperation data, i.e. teleoperation supplies exactly the mapping human demonstration lacks.
- **Force-supervised planner** — predicts **end-effector pose chunks *and* contact-force trajectories**.
- The predicted contact force becomes the **reference for a tactile-based admittance controller** — force is planned, then tracked by a compliant controller.

That closes the loop from demonstration to execution in force space, rather than only in position space.

## 4. Results

Five contact-rich tasks spanning **deformable objects, bulky rigid objects, and human–humanoid collaboration**. WT-UMI improves success rate and **reduces contact-position tracking error** over four policy baselines.

## 5. Stated limitations, all three substantive

1. **Coverage** — *"only the palms, forearms, and chest are instrumented"*; extending to dexterous hands, legs and back would broaden the task set.
2. **No vision** — *"our policy does not yet consume RGB vision input, because third-person views introduce a human–humanoid embodiment gap and finding a camera angle that consistently avoids object occlusion is challenging, especially for transport tasks."* An honest account of why a modality was omitted rather than a claim that it was unnecessary. Fusing vision *"would enable the policy to anticipate future contact and re-establish lost contact."*
3. **Scalar force only** — *"the force head predicts only a scalar normal force; extending it to multi-axis contact wrenches and distributed force maps would support finer force regulation."*

## 6. What it adds that the others don't

**Force as a first-class planned signal.** Almost every policy here consumes contact and emits positions; WT-UMI predicts a **contact-force trajectory** and hands it to an admittance controller as a reference — making force something the planner is responsible for rather than an emergent consequence. Together with [[twins]] it defines the whole-body tactile setting, and its shared human/teleoperation interface is a neat resolution of a trade-off the demonstration-collection cluster otherwise lives with.
