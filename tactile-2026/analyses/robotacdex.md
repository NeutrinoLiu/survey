# RoboTacDex — A Dexterous Visual-Tactile-Action Dataset for Humanoid Manipulation

**arXiv:2606.31836** · Fudan + ByteDance Intelligent Creation + Shanghai AI Lab + CUHK (X. Wang, D. Li, Z. Chen, C. Yu, C. Xin, P. Ye, Sun, T. Chen) · Jun 2026 · IEEE RA-L

**One line.** A humanoid (Unitree G1) tactile dataset whose most useful result is a **negative** one: adding camera views does nothing for existing imitation-learning policies, while adding tactile does.

## 1. What "tactile" means here

Fingertip tactile on a **Unitree G1** dexterous humanoid, recorded alongside **multi-view RGB and depth**, proprioception, and semantic annotations. The official Unitree teleoperation script was **re-engineered** to record depth and tactile at all.

The synchronisation work is a real contribution — a hardware-software co-synchronisation scheme achieving **millisecond alignment** across heterogeneous sensors, which is the practical obstacle to any multimodal humanoid dataset.

## 2. Dataset

| | |
|---|---|
| Trajectories | **6,000+** |
| Duration | ~**25 hours** |
| Tasks | **19** |
| Skills | **23** |
| Objects | **22** |

Tasks are *"relatively challenging... that can only be completed by dual arms and dexterous hands, aiming to mimic human-like operational logic."* Replayable both on the real robot and in **IsaacSim**.

The motivating gap is specific: data collection for fixed-base manipulators is mature, but *"high-quality manipulation datasets specifically for humanoid robots remain extremely scarce"*, because humanoids have very high DOF and demand data diversity for unstructured settings.

## 3. Findings

Three imitation-learning policies (**ACT, Diffusion Policy, GR00T**) evaluated across observation configurations.

**The multi-view null result.** On PickAndPlacePear and InsertBook, *"no model demonstrates significant performance improvement after adding wrist cameras or third-person perspective camera."* The diagnosis: *"For ACT and DP, the models lack mechanisms for multi-view image matching or additional supervision. Consequently, simply concatenating images from multiple viewpoints proves ineffective."* And *"the benefits of multi-view observations are not automatically realized by existing manipulation learning frameworks, but rather depend on the model's ability to reason about geometric relationships across viewpoints."*

For GR00T, single-view success is already high on the simple task, so extra views cannot help.

**Tactile, by contrast, helps** — and the mechanism is stated in the right terms: *"it reveals latent state variables that are visually ambiguous or entirely hidden, especially when the visual scene remains static but the physical contact state changes dramatically."* Outcome categories are reported (success / stuck / adjusts / failure), which is more diagnostic than a bare success rate.

The pairing of these two results is the paper's value: **more cameras is not more information to a policy that cannot fuse them, while touch is information no camera provides.** That is a cheap, generalisable lesson for anyone deciding what to instrument.

## 4. Stated limitations

*"Tactile information is incorporated in a relatively straightforward manner, without dedicated architectural components for multimodal fusion"* — so the reported tactile gains are a **lower bound**, achievable with naive fusion. And the dataset is single-environment; scale and environmental diversity remain to be extended.

## 5. What it adds that the others don't

A **humanoid**-specific tactile dataset with millisecond multi-sensor synchronisation on a widely available platform (Unitree G1), plus the multi-view null result. Most tactile datasets here are gripper- or fixed-arm-based; RoboTacDex is one of the few targeting the dual-arm dexterous humanoid setting that the industry is converging on. Compare [[deco]] (bimanual dexterous, 50 h) and [[hcttp]] (human-side, transferred to humanoid hands).
