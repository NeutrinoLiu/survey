# OmniTacTune — Policy-Agnostic Real-World RL for Tactile Residual Adaptation of Visual Policies

**arXiv:2607.03723** · University of Maryland + Georgia Tech (K. Yu, H. Zhang, Ravichandar, Y. Han, R. Gao) · Jul 2026 · [site](https://colinyu1.github.io/omnitactune-site/)

**One line.** Adds touch to *any* pretrained visual policy by real-world RL on a residual — **5–40% → 85–100% success in 40–80 minutes**, with no offline tactile demonstrations at all.

## 1. The framing

Visual policies from human video, teleoperation and robot data give scalable motion priors but *"often fail in contact-rich manipulation, where success significantly depends on local force and contact geometry."* Meanwhile tactile datasets *"remain orders of magnitude smaller than visual datasets"* and are *"hard to generalize across sensors, robots, and tasks."*

So rather than train a tactile policy, **adapt an existing visual one**, using its motion prior as the scaffold.

## 2. Method — two stages, no tactile demonstrations

1. **Warm-start** tactile-aware critics and encoders from **autonomous base-policy rollouts** — the base policy generates its own training data.
2. **Online RL** of a **lightweight tactile residual policy** through real interaction, with **multisensory reward shaping** (flow-derived reward, tactile reward).

## 3. The generality claim — and it is well tested

Policy-agnostic across three axes simultaneously:

| Axis | Coverage |
|---|---|
| **Base policies** | π₀.₅, Diffusion Policy, ACT — trained from human videos *or* robot data |
| **Tactile representations** | tactile images, tactile markers, high-res, low-res, dense, sparse |
| **Tasks** | Peg-in-Hole, Charger Insertion, Cap Opening, Box Opening |

Testing across base policies *and* tactile representations *and* tasks is what earns the "policy-agnostic" label; most residual methods fix two of the three.

## 4. Results

**5–40% → 85–100%** across four real contact-rich tasks, in **40–80 minutes** of online interaction per task.

The efficiency figure is the headline: under 90 minutes of real-robot RL converts a mostly-failing visual policy into a mostly-succeeding one, without collecting a single tactile demonstration.

They also note that **Sparsh and similar encoders are not pretrained on dynamic manipulation tactile data**, which limits what off-the-shelf tactile representations contribute — consistent with [[anytouch2]]'s tier analysis and [[geotlm]]'s chance-level rotation result.

## 5. Stated limitations

*"OmniTacTune still inherits common limitations of real-world RL, including the need for **manual resets** and **hardware wear under repeated contact-rich interactions**, especially when using fragile vision-based tactile sensors."* Proposed remedy: world models to generate physically plausible visuo-tactile rollouts for pretraining and online improvement — which is what [[taco]], [[vitacworld]] and [[taccorl]] do.

## 6. What it adds that the others don't

**Real-world RL that is cheap enough to actually run.** [[taccorl]] needs a digital twin, [[torl-vla]] needs human intervention, [[vitar]] needs restored decision points for preference labelling; OmniTacTune needs 40–80 minutes and a base policy. The residual formulation over a *frozen* visual policy also means the motion prior cannot be damaged — the same structural safety [[vitar]] argues for, obtained here through RL rather than bounded selection.
