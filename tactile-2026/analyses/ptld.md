# PTLD — Sim-to-Real Privileged Tactile Latent Distillation for Dexterous Manipulation

**arXiv:2603.04531** (v3, Jun 2026) · CMU + University of Washington + **FAIR at Meta** + UC Berkeley (R. Chen, Mukadam, T. Wu, Hogan, Kaess, Malik, Sharma) · Mar 2026 · [site](https://akashsharma02.github.io/ptld-website/)

**One line.** Trains tactile dexterous policies **without simulating tactile at all** — RL on a *privileged* sensor in simulation, then a real tactile encoder distilled to reproduce that privileged policy's latent state.

## 1. The dilemma it dissolves

Imitation learning needs demonstrations, but *"reliable teleoperation for intricate tasks like using a screwdriver, a wrench, or turning a doorknob is difficult"*, and kinesthetic teaching *"is equally challenging when more than two fingers are required."* RL avoids demonstrations but *"the lack of fast and realistic tactile simulation remains a core issue."*

PTLD's move: anchor sim-to-real on a **privileged sensor** (e.g. object pose) that simulation *can* produce faithfully, and learn a mapping from real touch into that policy's latent — **bypassing tactile simulation entirely**.

## 2. Pipeline

1. **RL in simulation** on privileged sensor observations → privileged policy
2. **Deploy in a real-world instrumented cell** to collect tactile demonstrations
3. **Train a tactile encoder** to map real touch into the privileged encoder's latent state
4. **At deployment**, the policy uses *only tactile and proprioception* — no external pose tracking

The last point is what makes it practical: *"making it robust to object property changes, wrist orientation changes, and generalizing to irregular geometries such as cube."*

## 3. Results

| Task | Improvement over proprioceptive-only sim policy |
|---|---|
| In-hand rotation (benchmark) | **+182%** |
| Tactile in-hand reorientation | **+57%** goals reached |

## 4. The design analysis — three named considerations

Unusually, the paper states the conditions under which its approach works:

- **Information asymmetry.** *"While a tactile sensor captures rich dynamics directly via contact patches and forces, object poses capture the effect of forces indirectly as kinematic data. **Selecting a privileged sensor that minimizes information asymmetry is critical for high-fidelity distillation.**"* If the privileged sensor cannot express what touch measures, the student cannot be distilled into it.
- **Privileged sensor noise floors.** *"High noise in object pose estimation... sets a performance ceiling on the deployed policy."* The teacher's sensing quality bounds the student.
- **Instrumented setup requirement.** External cameras or trackers are needed during distillation, which *"may limit rapid application of this method to completely unstructured or 'in-the-wild' environments."*

## 5. What it adds that the others don't

**Removing tactile simulation from the loop entirely.** The whole simulation cluster — [[tacauchy]], [[etac]], [[tacmap]], [[dot-sim]], [[tac2real]], [[hydroshear]] — exists to make simulated touch faithful enough to train on. PTLD asks whether that is necessary, and answers no: simulate something you *can* simulate, then bridge to real touch by distillation on real data. [[tacblr]] arrives at the same conclusion independently and with unpaired data.

The information-asymmetry caveat is the honest limit: this works when a privileged proxy exists that captures most of what touch would contribute, and in-hand rotation — where object pose largely determines the task — is close to the best case.
