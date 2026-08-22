# OmniUMI — Towards Physically Grounded Robot Learning via Human-Aligned Multimodal Interaction

**arXiv:2604.10647** (v3, May 2026) · **BAAI** + CASIA + UCAS + BIT + BUPT + Peking University (S. Luo, Y. Li, Hu, C. Yu, C. Xu, J. Zhang, Yao, T. Huang, R. He, Z. Wang) · Apr 2026 · [site](https://baai-aether.github.io/OmniUMI)

**One line.** Extends UMI from visuomotor to **fully physical**: RGB, depth, trajectory, tactile, **internal grasping force**, and **external interaction wrench** in one handheld device — with **bilateral gripper feedback** so the human can feel and modulate the forces they are demonstrating.

## 1. The gap

*"The current evolution of UMI-like systems remains largely confined to visuomotor data... which are effective for geometric manipulation but provide only limited access to the physical variables that govern contact-rich behavior."*

The consequence, enumerated: *"a broad class of tasks — including wiping, screwing, deformable object handling, compliant assembly, and fragile grasping — remains difficult to learn from vision-dominant data alone."*

Prior extensions added *either* tactile *or* force/torque; OmniUMI captures both, plus the distinction between them.

## 2. The three contact quantities

Worth separating, because the paper is one of the few to treat them as different things:

| Quantity | What it measures |
|---|---|
| **Tactile sensing** | local contact geometry and distribution at the fingertip |
| **Internal grasping force** | how hard the gripper squeezes the object |
| **External interaction wrench** | forces exchanged between object and environment |

A fragile grasp is an internal-force problem; wiping is an external-wrench problem; selective release is a tactile problem. Vision gives none of them.

## 3. System

Compact handheld device capturing all six streams synchronously, with **collection–deployment consistency through a shared embodiment design**.

The **human-alignment** mechanism is the distinctive part: **bilateral gripper feedback** plus the handheld embodiment enable *"natural perception and modulation of internal grasping force, external interaction wrench, and tactile interaction"* — the demonstrator does not merely produce force data, they *feel and regulate* it.

**Learning and deployment:** diffusion policy extended with visual, tactile and force observations, deployed through **impedance-based execution** for *"unified regulation of motion and contact behavior."* Matching the policy's output space to a compliant controller is what lets learned force intent actually execute.

## 4. Results

Reliable sensing and strong downstream performance on three deliberately chosen tasks, one per contact quantity:
- **force-sensitive pick-and-place** (internal force)
- **interactive surface erasing** (external wrench)
- **tactile-informed selective release** (tactile)

## 5. Stated limitations

Evaluated on *"a limited set of contact-rich tasks and a single system instantiation"*, with *"a more comprehensive analysis across operators, embodiments, and task families"* still needed. They explicitly call for **controlled ablations and user studies** on human alignment — *"finer-grained analysis of how bilateral feedback and handheld embodiment jointly shape internal-force, external-force, and tactile interaction"* — which is the study [[vihateleop]] partially runs.

## 6. What it adds that the others don't

**The most complete physical instrumentation of a handheld interface**, and the separation of internal force from external wrench from tactile — three quantities the rest of this survey usually collapses into "contact." Its closing observation is the right summary of the whole data-collection cluster: *"progress in contact-rich robot learning depends not only on stronger policy models, but also on better interfaces for acquiring, aligning, and deploying physically meaningful interaction data."*
