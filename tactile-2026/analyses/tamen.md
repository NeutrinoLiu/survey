# TAMEn — Tactile-Aware Manipulation Engine for Closed-Loop Data Collection in Contact-Rich Tasks

**arXiv:2604.07335** · Fudan + Shanghai Innovation Institute + **OpenDriveLab (HKU)** + SJTU + ECUST (L. Wu, Ren, C. Jiang, J. Zhou, Peng, R. Huang, L. Chen, H. Li) · Apr 2026 · [site](https://opendrivelab.com/TAMEn)

**One line.** Turns handheld data collection into a **closed loop**: collect demonstrations, run the policy, then use AR-based teleoperation *with tactile feedback* to collect recovery data at the exact states where the policy failed.

## 1. Three gaps in handheld collection

1. **Hardware adaptability** — prior designs are gripper-specific.
2. **Precision–portability trade-off** — high tracking precision requires fixed infrastructure.
3. **No feasibility checking** — demonstrations turn out not to be replayable on the robot.
4. And most importantly: *"existing handheld setups struggle to collect interactive recovery data during robot execution, lacking the authentic tactile information necessary for robust policy refinement."*

## 2. System

**Wearable visuo-tactile interface** adapting rapidly across heterogeneous grippers, with a fisheye camera, markers, visuo-tactile sensor, linear guideway and gripper-opening sensing.

**Dual-mode acquisition** breaks the precision–portability trade-off:
- **Precision mode** — motion capture, **sub-millimetre** fidelity
- **Portable mode** — VR/ArUco tracking, for in-the-wild acquisition **and** tactile-visualised recovery teleoperation

**Online feasibility checking** during demonstration, so what is recorded is replayable.

**tAmeR** — an AR-based teleoperation system with egocentric observation, used to collect recovery data **with tactile feedback during policy execution**, feeding back into training.

## 3. The data pyramid

Three tiers for staged learning: **large-scale tactile pretraining** → **task-specific bimanual demonstrations** → **human-in-the-loop recovery data**, targeting generalisation, coordination and failure recovery respectively.

Training mirrors it: tactile representation pretraining (contrastive) → ACT imitation on bimanual demos → recovery-based refinement. Per-task action chunk sizes, lr 1e-5, KL weight 10, trained on trajectory sequences rather than sampled frames.

The **recovery-data taxonomy is task-specific and published**, which is unusually concrete:
- *Herbal transfer* — failed grasping and pouring
- *Cable mounting* — failing to grasp the cable, dropping during transport/insertion, misaligned placement
- *Binder clip removal* — grippers failing to establish simultaneous grasp, unsuccessful clip grasping
- *Dish washing* — missing the sponge, missing the dish, requiring repeated wiping

## 4. Results

Average success across diverse bimanual tasks: **34% → 75%**. Feasibility-aware collection *"significantly improves demonstration replayability."*

## 5. What it adds that the others don't

**Recovery data collected with real tactile feedback at real failure states.** Three papers in this survey attack the missing-near-failure-data problem — [[taccorl]] simulates it, [[taco]] imagines it with a world model, [[torl-vla]] learns from human takeovers online — and TAMEn *teleoperates it*, with the operator feeling contact through AR. That produces authentic contact signals at failure states, which neither simulation nor generation can guarantee. The dual-mode precision/portable pipeline and online feasibility checking are also practical fixes to problems every UMI-derived system has.
