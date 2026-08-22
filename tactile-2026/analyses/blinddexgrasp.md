# Blind Dexterous Grasping via Real2Sim2Real Tactile Policy Learning

**arXiv:2606.11767** (v2) · ShanghaiTech + BIGAI (S. Luo, X. Huang, Z. Xu, W. Li, Jiao, Xiao) · Jun 2026 · [site](https://Dex-Blind-Grasp.github.io)

**One line.** Tactile-only grasping with **no vision at all**, transferred zero-shot from a contact-calibrated digital twin — and it reports its **27%** real success rate without inflation.

## 1. The setting

*"Blind grasping is essential for human and robot manipulation when vision is unreliable, occluded, or unavailable... It is therefore well suited to darkness, clutter, narrow spaces, and severe occlusion."*

Two obstacles: *"the tactile sim-to-real gap and the **limited expressiveness of sparse tactile signals**."*

## 2. Three components

1. **Real2Sim tactile calibration** — building a *"contact-calibrated digital-twin simulator capable of reproducing real tactile signals."*
2. **Layout-aware tactile encoder** — improving expressiveness of sparse tactile by *"incorporat[ing] sensor-geometry priors through self-supervised pretraining."* Where the taxels physically sit is prior information the encoder should not have to learn.
3. **RL experts → Diffusion Policy** — train **object-specific** RL experts in the calibrated simulator, then aggregate their successful grasp trajectories into a single **tactile-conditioned Diffusion Policy** for generalisation.

## 3. Results

Physical **LEAP Hand** with distributed tactile, **10 seen and 10 unseen objects**: **27% real-world grasp success across all 20**, with **no real-world grasping demonstrations and no visual input**.

The behaviour sequence is the interesting part — *init → search → contact → adjust → lift* — a policy that **dynamically searches for the object**, adjusts grasp contacts, and lifts, using only robot state and sparse tactile.

**Ablations:** layout-aware tactile pretraining improves grasping; sensing-level evaluation confirms Real2Sim calibration *"increases the consistency of **contact events** between simulation and hardware."* Contact *events*, not signal values — the same choice of alignment target as [[vibeact]].

## 4. On the 27%

Reporting 27% as a headline is unusual and worth respecting. Blind grasping of arbitrary objects with sparse tactile is genuinely unsolved, and the paper says so, identifying what would fix it: *"richer tactile representations, **recovery and regrasp behaviors**, and **denser full-hand tactile sensing**."*

That third item is the recurring conclusion — [[tactilegenesis]] shows coverage dominates in simulation, [[t-rex]] names absent palm sensing as a hardware bottleneck, and here it bounds real performance.

## 5. What it adds that the others don't

**Vision-free operation as the design point**, with an honest number attached. Where [[htt]]'s camera-free experiments use a rich tactile hand and [[roto2]]'s blind policies live in simulation, this is sparse real tactile on real hardware with unseen objects — the hardest version of the setting, and a useful calibration of how far tactile-only dexterity currently reaches.
