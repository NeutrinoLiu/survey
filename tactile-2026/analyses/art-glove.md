# ART-Glove — Articulated Tactile Glove for Contact-Grounded Dexterous Interaction Capture

**arXiv:2606.16370** · Carnegie Mellon University (Changyi Lin, Ding Zhao) · Jun 2026 · [site](https://linchangyi1.github.io/ART-Glove)

**One line.** Replaces the soft deformable glove with **16 rigid functional surfaces on 22 anatomically aligned joints**, so hand-side contact geometry is *known* rather than estimated — the single design choice that makes a demonstration physically interpretable.

## 1. The argument

The problem with hand kinematics alone: *"the same motion can produce different outcomes depending on **which hand-side surface contacts the object**, the local surface geometry, how that surface moves, and when and where contact occurs."*

So a useful demonstration must provide *"known or reliably recoverable hand-side contact geometry, tracked surface motion, and measured or inferred contact state."*

Their comparison table is the clearest taxonomy of dexterous capture methods in this survey:

| Method | Motion freedom | Haptic feedback | Contact geometry | Surface motion | Tactile |
|---|---|---|---|---|---|
| Teleoperation | robot-constrained | indirect/limited | known robot geometry | robot state | robot-dependent |
| Exoskeleton | constrained | surface-mediated (fingertips) | known device geometry | encoder (12-DoF) | sparse/optional |
| Passive-hand linkage | constrained | linkage-mediated | known robot geometry | encoder (12-DoF) | varies/optional |
| Bare-hand video | **free** | direct | **estimated deformable hand** | vision (21-DoF) | none/inferred |
| Soft sensing gloves | near-free | near-direct | **estimated deformable glove** | glove-dependent | sparse/optional |
| **ART-Glove** | **near-free** | surface-mediated | **known rigid surfaces** | **encoder (22-DoF)** | **dense (2048 taxels)** |

The two "estimated deformable" rows are the point: video and soft gloves free the hand but leave contact geometry uncertain. ART-Glove keeps the hand nearly free *and* makes geometry exact, by making the contacting surfaces rigid.

## 2. Hardware

- **16 rigid functional surfaces** covering fingers, thumb and palm
- **22 anatomically aligned joints** connecting them, following natural hand motion
- **Encoder-based** surface motion tracking → 22-DoF
- **Dense piezoresistive tactile** over the same surfaces → **2048 taxels**
- Synchronised capture at **120 Hz**

Evaluated on motion freedom, joint sensing, tactile sensing, and contact-rich interaction capture.

## 3. The transfer story

The proposed representation is worth quoting, because it reframes human-to-robot transfer:

> *"This description represents demonstrations through **hand-side surface properties**, including known geometry and nominal material, together with surface motion and contact state, rather than only joint trajectories. A robot hand therefore does not need to exactly match any individual glove; it should instead provide comparable information about its own contact surfaces, their motion, and contact state."*

And the consequence: *"learning can move from one-to-one joint matching toward **modeling physical interaction between moving surfaces**."*

That is a genuinely different answer to the embodiment gap than retargeting joints ([[hrdexdb]]), aligning latents ([[tactalign]]), or unifying token spaces ([[ftp-1]]) — it says the transferable unit is a *surface*, not a joint.

## 4. Stated limitation

**One hand size.** Scaling requires personalising *both* mechanical structure and sensing layout: *"the rigid surface links should match each user's phalanx and palm geometry, the joint axes should remain aligned with the user's hand motion, and the tactile FPCBs should preserve consistent taxel-to-surface mappings across customized gloves."* Proposed: generate designs from smartphone or portable-3D-scanner hand scans, auto-producing fabrication files for shells, joints, FPCBs and calibration tools.

## 5. What it adds that the others don't

**Making hand-side contact geometry a known quantity by construction.** Every other human-side capture device in this survey trades either dexterity or geometric certainty; ART-Glove's rigid-functional-surface decomposition gets both, at the cost of per-user fabrication. Its surface-level transfer formulation is also the most conceptually clean proposal here for what should actually be shared between a human hand and a robot hand.
