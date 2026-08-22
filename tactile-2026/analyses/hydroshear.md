# HydroShear — Hydroelastic Shear Simulation for Tactile Sim-to-Real Reinforcement Learning

**arXiv:2603.00446** · University of Michigan + **Amazon Industrial Robotics** (Dang, J. Lee, Mukadam, X. A. Wu, Nambi, Bucher, Fazeli) · Feb 2026 · [site](https://hydroshear.github.io)

**One line.** Argues the tactile-simulation field has optimised the wrong thing — **image rendering quality instead of shear physics** — and shows a policy trained on accurate shear beats one trained on tactile images by **59 points**.

## 1. The critique

*"Existing methods primarily focus on vision-based sensors and emphasize **image rendering quality while providing overly simplistic models of force and shear**. Consequently, these models exhibit a large sim-to-real gap for many dexterous tasks."*

## 2. Method

A **non-holonomic hydroelastic** tactile simulator modelling three things prior work does not:
- **stick-slip transitions**
- **path-dependent force and shear build-up** (hence *non-holonomic* — the shear state depends on the history of contact, not just the current configuration)
- **full SE(3) object-sensor interactions**

It extends hydroelastic contact using **SDFs to track displacements of on-surface points of an indenter** during interaction with the sensor membrane, producing *"physics-based, computationally efficient force fields from arbitrary watertight geometries while remaining agnostic to the underlying physics engine."*

## 3. Tasks — each isolating a different tactile challenge

The task design is unusually deliberate:
- **(a) Peg insertion** — reasoning about **in-hand pose uncertainty** during tight insertion
- **(b) Bin packing** — **simultaneous or sequential multi-object contacts** through pushing to insert
- **(c) Book shelving** — lateral insertion with **gravity orthogonal to the insertion axis**, with full contact patches at the fingertips *"that make object features such as edges less informative for pose inference"*
- **(d) Drawer pulling** — precise, **minimal gripper force modulation** under unknown perturbations that induce slippage

## 4. Results

| Policy training signal | Average success |
|---|---|
| **HydroShear** | **93%** |
| Alternative shear simulation methods | 58–61% |
| **Tactile images** | **34%** |

Zero-shot sim-to-real across all four tasks. The 34% row is the paper's argument in one number: **training on simulated tactile *images* is far worse than training on simulated *shear*, even though images are what the field has spent most of its simulation effort rendering.**

The keyframe analysis shows the behaviours this buys: reactive repositioning after first extrinsic contact, "unsquish" corrections before insertion, and **adaptive grasp modulation to prevent further slippage** under random force perturbation during drawer pulling.

## 5. What it adds that the others don't

**Shear as the quantity to simulate faithfully.** This is the sharpest challenge in the simulation cluster to what the rest of it optimises: [[tacmap]] targets penetration depth, [[dot-sim]] targets optical realism, [[tacauchy]] targets stress fields, and HydroShear argues that for policies, **path-dependent shear and stick-slip** are what determine transfer. Its 93% vs 34% comparison against image-trained policies is the strongest single piece of evidence in this survey that tactile image fidelity and tactile *usefulness* are different objectives — consistent with [[tactile-wam]]'s pixel–contact misalignment and [[tacgen]]'s 13-point reconstruction-vs-utility gap, here measured in policy success.
