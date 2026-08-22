# TacCoRL — Integrating Tactile Feedback into VLA via Simulation

**arXiv:2606.11743** · UCLA + UCSD + UESTC + Peking University + Utah (S. Ma, Y. Liang, C. Yu, Y. Chen, H. Su, Y. Zhu, Y. Yang, C. Jiang) · Jun 2026 · [site](https://tac-corl.github.io/)

**One line.** Solves the near-failure-data problem: the states where touch matters most are *rare in demonstrations and dangerous to collect on hardware*, so learn them in a real-aligned simulator with RL, anchored by a small real dataset.

## 1. What "tactile" means here

Tactile encoded by a **CNN encoder** and routed through **contact-aware gating** that modulates **both** the VLM context and the action expert — a two-point injection, unlike the single-point designs in most of this cluster.

## 2. The argument

Real demonstrations are structurally the wrong data for this problem: *"They are costly to scale, emphasize successful nominal behavior, and underrepresent the near-failure cases where tactile feedback is most critical: slight misalignment, contact on the wrong surface, and unstable grasp–object interactions."*

And imitation learning cannot fix it: it exposes the policy to tactile *observations* but *"provides limited supervision on how contact should influence actions in off-nominal states."* Collecting those states on hardware *"risks sensor damage and incurs slow and costly resets."*

Hence the requirement: a **safe, resettable, verifiable simulated tactile environment** where contact conditions can be systematically varied and learned in closed loop. The key framing is that the contribution is *"not only adding touch as an input, but learning how contact readings should modulate action responses."*

## 3. Data curriculum — three sources, three stages

**(A) Collection.** Real teleoperation `D_real`, simulated teleoperation `D^teleop_sim`, then scaled with **MimicGen** into `D^Mimic_sim`. The simulator is aligned to the real setup through **matched scene configuration, robot controller response, and tactile-contact interface calibration** — a task-level digital twin.

**(B) Sim-real co-training** gives the pretrained VLA an initial tactile-conditioned action prior:
```
L_SFT = α·L^sim_SFT + (1−α)·L^real_SFT
```

**(C) RL fine-tuning** with sparse verifiable task rewards in simulation, anchored by real data:
```
L_RL = L_PPO + β·L^real_SFT
```

The policy transfers **directly to the real robot without privileged simulation state or online real-world RL**.

## 4. Results

Four bimanual contact-rich tasks — **Assembly #1, Assembly #2, Test Tube Insertion, Do Puzzle** — real-world success:

| Stage | Vision-only: A#1 / A#2 / Tube / Puzzle | Visuo-tactile: A#1 / A#2 / Tube / Puzzle |
|---|---|---|
| Real-only fine-tuning | 0.20 / 0.05 / 0.35 / 0.25 | 0.45 / 0.15 / 0.35 / 0.40 |
| Sim-real co-training | 0.35 / 0.10 / 0.40 / 0.35 | 0.50 / 0.25 / 0.45 / 0.55 |
| **RL post-training** | 0.35 / 0.25 / 0.80 / 0.60 | **0.70 / 0.45 / 0.95 / 0.80** |

Average **50.0% → 72.5%** from adding tactile *on top of* RL post-training — the authors note this specifically indicates *"the gain comes from contact sensing rather than from simulator RL alone."* Comparing the two RL rows is the right control, and it is the one they run.

**Do Puzzle** is the longest-horizon task (all three pieces must be placed). Real-only fine-tuning and sim-real co-training *"often stop after partial completion,"* while RL post-training reinforces complete trajectories: **15% → 25% → 45%** (vision-only) and up to **80%** with tactile.

**Hyperparameter ablation** (Assembly #2) — unusually informative because it reports simulator success, real-anchor loss, *and* real deployment success together:

**Co-training ratio α** (with β = 0):
| α | Best simulator success | Real deployment |
|---|---|---|
| 0.95 (mostly sim) | 42.9% | 40% |
| 0.5 | **70.3%** | **45%** |
| 0 (no sim demos) | 14.1% | 25% |

The α = 0 row is the informative one: starting RL from a real-only policy in zero-shot simulation fails badly, because *"the initial simulator rollouts are too far from the policy's learned interaction distribution for effective post-training."* Sim-real co-training is not optional pre-conditioning — it is what makes simulator RL reachable at all.

**Anchor weight β** (with α = 0.5):
| β | Best simulator success | Real deployment |
|---|---|---|
| 0 | 70.3% | 45% |
| 0.1 / 1.0 | **>92%** | **80%** |
| 5.0 | lowest supervised loss, suppressed RL | reduced |

Both moderate settings nearly double real deployment success (45% → 80%). Without the anchor, *"the policy can overfit to simulator-specific contact strategies."* With too strong an anchor (β = 5.0), RL refinement is suppressed and the policy is pulled back toward imitation, **reducing both simulator and real success**. A clean two-sided optimum, rarely shown this explicitly.

**Qualitative:** the post-trained policy *"uses tactile feedback to correct local contact errors through incremental translations and reorientations"* — the corrective behaviour is visible, not just inferred from success.

## 5. Stated limitations, all substantive

- **Still needs real tactile data** as an anchor, though far less of it.
- **Simulation setup effort** — a task-level digital twin requires manual asset reconstruction, camera alignment and tactile system identification.
- **Hard-to-simulate contacts** — experiments are confined to rigid or near-rigid contact where the simulator gives reliable contact predicates and tactile statistics; deformables and fluids need modelling beyond the current setup.

That third point bounds the whole approach, and marks the boundary between this line and the deformable-object work ([[softvtbench]], [[deform360]]).

## 6. What it adds that the others don't

The **near-failure-data argument** and a working pipeline for it. Every other tactile-VLA in this survey trains on demonstrations of success and hopes the policy generalises to contact errors; TacCoRL manufactures the errors safely and learns the corrections. The **two-sided β ablation** — too little anchor overfits the simulator, too much suppresses RL — is the most useful practical result for anyone doing sim-based post-training, and the **α = 0 collapse** shows why sim-real co-training must precede simulator RL rather than replace it. Compare [[torl-vla]] (online RL on the real robot with human intervention) and [[taco]] (imagined corrections from a world model) — three different answers to the same missing-data problem.
