# TactiDex — A Real-World Tactile-Guided Benchmark for Human-Like Dexterous Manipulation

**arXiv:2607.09190** · ShanghaiTech University (Ni, Zhang, Wei, Chen, Zhang, Shi, Wang) · Jul 2026 · [tactidex.github.io](https://tactidex.github.io/)

**One line.** A whole-hand human tactile dataset whose contact forces are used not as policy *input* but as an RL *reward*, on the argument that kinematic retargeting produces air-grasping robots that look right and touch wrong.

## 1. What "tactile" means here

Tactile is **whole-hand normal pressure from a human hand**, not a robot sensor. The capture rig is a dual-glove sandwich:

- **Outer tactile glove** — piezoresistive, **162 sensing elements** across fingertips and palm, force resolution to **0.01 N**, sampling at **17 Hz**. The authors redesigned the glove's structure so it can be worn *over* a mocap glove without degrading kinematic tracking.
- **Inner capture glove** — IMU + optical fusion for articulated joint states.
- **OptiTrack**, 8× PX13W cameras at **120 fps** over a 1.2 m × 1.8 m volume, giving 6-DoF wrist and object pose in one global frame.

The three streams are hardware-synchronised through an eSync module, so tactile pressure is frame-aligned with the 120 Hz kinematic stream. Note the rate mismatch is real and unhidden: touch is the slowest stream at 17 Hz.

Crucially, tactile is treated as a **measurement of force modulation** — the thing the authors argue geometric datasets can only *infer* from proximity. On the robot side, contact force is whatever the simulator reports at the fingertips; the human pressure map becomes the target.

## 2. Data curriculum

| | |
|---|---|
| Frames | **5.1 M** |
| Sequences | **757** interaction sequences |
| Objects | **49** calibrated, 3D-scanned |
| Coverage | single-hand and bimanual; pick-place → tool use → functional operations (click, cut, shake, drink, cook) |
| Annotations | interaction-phase timestamps, task identifiers, natural-language descriptions |

Object meshes come from mobile 3D scanning + SAM3D reconstruction, with non-reflective tape and fiducials applied to defeat feature-matching failure on shiny/transparent/symmetric surfaces.

The processing pipeline is the interesting part, because **tactile is used to clean the kinematics**:

1. MANO fitted to raw mocap keypoints by L-BFGS, shape shared across frames per subject, with an ℓ2 pose regulariser and a second-order temporal smoothness penalty.
2. **Two-stage tactile-constrained post-optimisation.** Genuine contact intervals are identified by gating geometric proximity *with measured pressure* — this filters "visually deceptive near-contacts". Inside each interval a stable reference grasp frame is chosen and aligned using tactile-modulated per-finger weights: fingers above the pressure threshold are strongly attracted to the object surface, inactive fingers are down-weighted to prevent phantom contacts.
3. The refined grasp is propagated across the interval by temporal IK, then a collision-aware pass optimises finger articulation against the object SDF to remove residual penetration.

So the dataset's own annotation quality depends on touch. That is a genuinely different use of the modality from every policy paper in this survey.

## 3. Model

**TactiSkill** is residual RL, not a learned perception stack:

- π_base — a **frozen kinematic imitation policy** (built on ManipTrans-style morphological retargeting).
- π_res — a learnable residual whose action is Δq_t, residual joint position commands.

The authors' framing: π_res is repurposed from a *geometric compensator* into a **force modulator**. There is no tactile encoder, no tactile tokens, no visual backbone. The whole contribution sits in the reward.

## 4. How tactile enters the model

Not as an encoded observation stream. Tactile enters in **two** places, both unusual:

- **State space** — a *Target Tactile Reference* `F_t^human`, the per-finger normal force the human applied at that moment, is appended to proprioception (q, q̇) and object pose. This is a reference signal, i.e. a feed-forward target, not a sensor reading.
- **Reward** — the **Tactile-Guided Tri-Component** objective:
  1. **contact guidance / bonus** — encourage meaningful and timely contact formation;
  2. **human-like alignment** — match the *distribution* of force across fingers to the human demonstration;
  3. **safety constraint** — regularise excessive force.

The authors are explicit about why the reward is decomposed: naïve contact-force matching alone produces unstable optimisation, reward exploitation, and physically implausible behaviour. The tri-component split is what makes tactile supervision trainable.

## 5. Experiment setup

- **Simulation**, 73 evaluation sequences, single-hand and bimanual.
- **Baseline** — ManipTrans, a purely kinematic transfer method.
- **Ablations** — w/o Bonus, w/o Align, w/o Safety.
- **Metrics**, and this is where the benchmark contribution really is:
  - `SR_kin` — kinematic success (OTE-r ≤ 30°, OTE-t ≤ 3 cm, MPJPE ≤ 8 cm, TipErr ≤ 6 cm)
  - `SR_tac` — *tactile-aware* success: kinematic criteria **and** MTFE ≤ 3.0 N **and** Contact F1 ≥ 0.3
  - `MTFE` — mean absolute per-finger force error vs. human ground truth
  - `Contact F1` — binarised contact states, spatio-temporal precision/recall of *which finger touches when*
  - `PeakSafe@3N` — fraction of episodes whose worst instantaneous force error stays under 3 N
  - `SafeTac@3N` — tactile success **and** peak safety
- **Real-world deployment** — dual 7-DoF Franka arms + Inspire hands. Simulated trajectories are retargeted to the Inspire's 6 independent actuators via an anchor mapping plus a fingertip-position + smoothness objective.

## 6. Does tactile actually help?

| Method | SR_kin ↑ | SR_tac ↑ | OTE-t (cm) ↓ | MTFE (N) ↓ | Contact F1 ↑ | PeakSafe@3N ↑ | SafeTac@3N ↑ |
|---|---|---|---|---|---|---|---|
| Kinematic base (ManipTrans) | 0.7291 | 0.3935 | 1.1947 | 0.1491 | 0.5569 | 0.6366 | 0.3584 |
| TactiSkill w/o Bonus | 0.7687 | 0.4193 | 1.2031 | 0.1529 | 0.5425 | 0.5700 | 0.3568 |
| TactiSkill w/o Align | 0.7637 | 0.4220 | 1.0828 | 0.1412 | 0.5134 | 0.5988 | 0.3441 |
| TactiSkill w/o Safety | 0.7705 | 0.4124 | 1.0265 | **0.0921** | 0.5686 | 0.6870 | 0.3872 |
| **TactiSkill (full)** | **0.8195** | **0.6464** | **0.9577** | 0.1236 | **0.7384** | **0.7280** | **0.5357** |

Read the two success columns together. The kinematic baseline looks fine at 72.9% — until you check *how* it succeeded: `SR_tac` collapses to 39.4%, meaning roughly **46% of its "successes" were air-grasping or mesh penetration**. Tactile supervision adds +9 points of kinematic success but **+25 points of tactile-aware success** and +0.18 Contact F1.

The most instructive row is **w/o Safety**, which posts the *best* MTFE in the table (0.0921 N) and the *worst* end-to-end reliability among the ablations on the metric that matters (SafeTac@3N 38.7% vs 53.6%). Without a penalty on transient spikes the policy games the mean: it keeps average force error low while permitting occasional destructive impulses. This is a clean demonstration that averaged tactile error is the wrong headline metric for contact safety — a point most of the policy papers in this survey do not test.

**Honest limits.** Everything quantitative is in simulation; the real-world section is qualitative. And the deployment has a telling gap the authors state plainly: *"Tactile sensing is available on the Inspire hands but is not used as feedback in the current deployment."* So the tactile loop is open at the point where it would matter most. The named limitation — sensor noise, resolution and compliance mismatch between sim and real degrading learned force interaction — is the honest version of that.

## 7. What it adds that the others don't

Three things. (a) Tactile as **reward** rather than observation, with an explicit account of why the naïve version fails. (b) Tactile as a **data-cleaning signal** — gating contact intervals by measured pressure to reject visual near-contacts. (c) A **success criterion that can fail a task the robot completed**, which is the same structural move [[softvtbench]] makes for deformables, arrived at independently.
