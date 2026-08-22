# AnyTouch 2 — General Optical Tactile Representation Learning for Dynamic Tactile Perception

**arXiv:2602.09617** · Renmin University + BAAI + BJTU + Shanghai AI Lab + CASIA + BUPT + PKU (Feng, Y. Zhou, Mei, D. Zhou, P. Wang, Cui, Fang, Yao, Hu) · Feb 2026 · [site](https://gewu-lab.github.io/AnyTouch2/) · [code](https://github.com/GeWu-Lab/AnyTouch2)

**One line.** Argues the field's tactile datasets are stuck at the bottom of a **five-tier dynamic pyramid** — mostly press-only, object-property data — and builds the missing top tiers, then shows the resulting model reveals a real **static/dynamic trade-off** in tactile pretraining.

## 1. The Tactile Dynamic Pyramid

The organising contribution, ordered by data rarity and the capability it supports:

| Tier | Data type | Capability | Existing coverage |
|---|---|---|---|
| **5** | Press only | object-level semantic properties (material, hardness) | abundant — TVL, VisGel, Touch and Go |
| **4** | Random action | basic dynamic contacts, initial sliding/rotating cues | some — TacQuad, YCB-Slide, Touch-Slide |
| **3** | Specific action | structured tactile dynamic semantics | **scarce** |
| **2** | Manipulation | temporally evolving contact patterns for dexterous skills | **scarce** |
| **1** | Force | tactile dynamics grounded in physical force | **scarce** — only FeelAnyForce (press-based) |

*"Most current datasets fall into the lower tiers (4 and 5), while higher tiers (1, 2, and 3) remain notably scarce."* This taxonomy alone reframes the tactile-data landscape more usefully than the usual scale comparisons.

## 2. Data curriculum — ToucHD

**2,426,174 contact samples**, built specifically to fill tiers 1–3:

| Subset | Contents | Tier |
|---|---|---|
| **ToucHD (Sim)** | simulated atomic actions — 5 sensors, 6 actions, 1,043 objects, **1,118,896 frames**; 3D objects × sensor backgrounds | 3 |
| **ToucHD (Mani)** | real-world manipulation via a modified **FastUMI** with tactile — 3 sensors, 46 tasks, **584,842 frames** | 2 |
| **ToucHD (Force)** | touch–force pairs from **71 indenters**, 5 sensors, **722,436 frames** | 1 |

## 3. Model — AnyTouch 2

Beyond the AnyTouch-1 objectives (masked video reconstruction, multi-modal alignment, cross-sensor matching), three **tier-targeted** additions:

- **Frame-difference reconstruction** → sensitivity to subtle temporal deformation (foundational dynamic objective)
- **Action matching** → semantic-level action understanding (Tier 3)
- **Temporal force prediction** from touch–force pairs → physical grounding (Tier 1)

## 4. Results — and one genuinely interesting negative

**Module ablation** (↓ marks significant drop):

| Removed | TAG Acc ↑ | Cloth Acc ↑ | Slip F1 ↑ / ΔF RMSE ↓ (DIGIT) | Force RMSE ↓ (DIGIT) | ToucHD Force RMSE ↓ (DIGIT) |
|---|---|---|---|---|---|
| **AnyTouch 2** | **76.97** | **42.31** | 86.66 / 87.80 | 624.26 | 894.32 |
| − Diff Recon | 76.19 | 41.33 | 84.39↓ / 94.88↓ | 687.13↓ | 1009.44↓ |
| − Action Match | 76.93 | 42.05 | 84.42↓ / 87.98 | 643.75 | 896.21 |
| − Force Pred | 76.46 | 41.45 | 86.35 / 90.72 | 770.44↓ | 1646.95↓ |
| **− MM Aligning** | **63.84↓** | **37.61↓** | **87.31** / **81.44** | **589.13** | 976.73↓ |
| − ToucHD (Sim) | 76.54 | 41.97 | 84.68↓ / 88.78 | 624.39 | 992.96↓ |
| − ToucHD (Mani) | 76.43 | 41.01 | 86.13 / 88.12 | 655.56 | 1118.49↓ |
| − ToucHD (Force) | 74.33↓ | 40.87↓ | 84.91↓ / 107.43↓ | 777.41↓ | 1792.49↓ |
| **− ToucHD (all)** | **68.92↓** | 40.39↓ | 84.16↓ / 110.68↓ | 783.64↓ | **2448.89↓** |

Each module degrades exactly the tier it targets — action matching → slip, force prediction → force, frame-difference → everything dynamic. That is a well-designed ablation.

**The negative result is the important one.** Removing **multi-modal alignment** *improves* most dynamic tasks (slip F1 86.66 → 87.31, ΔF RMSE 87.80 → 81.44, force RMSE 624 → 589) while collapsing object-level accuracy (76.97 → 63.84). The explanation: *"multi-modal alignment inherently emphasizes static tactile features, bringing together different possible actions on the same object, which can somewhat compromise the model's fine-grained dynamic perception."*

This is a **real trade-off between static object properties and dynamic tactile features**, and it indicts a large part of the tactile-representation literature — every touch-vision-language contrastive model (UniTouch, Touch100k, TVL) is optimising the objective that *hurts* dynamics. It also aligns with what [[taf-vla]] and [[restacvla]] argue from other directions: aligning touch to vision targets the shared part, not the useful part.

**Sensor comparison, a rarely-published observation:** GelSight Mini, with cleaner background and sharper deformation imaging, wins **Tier-5** (static, fine detail). DIGIT, with **30 Hz vs GelSight Mini's 18 Hz**, provides denser dynamic information and more training samples, and wins **higher-tier manipulation** tasks. Resolution buys static perception; bandwidth buys dynamic perception. Neither dominates.

**Real-world:** AnyTouch 2 outperforms all baselines across four real tasks, including the hardest **Tier-1 Chip Moving** task. MAE (S) trained only on lower-tier data reaches Tier-2 capability when ToucHD is added, but *"except accurate force perception for Tier 1"* — force grounding needs the force subset specifically.

## 5. What it adds that the others don't

The **pyramid** as a diagnostic frame for tactile datasets, and the **static-vs-dynamic trade-off** as a measured phenomenon rather than an intuition. Together they explain why so many tactile encoders that score well on material classification transfer poorly to manipulation: they are trained on Tier-5 data with a Tier-5 objective. The 30 Hz vs 18 Hz sensor finding is a useful practical note for anyone choosing hardware, and the touch–force subset from 71 indenters complements [[taf-vla]]'s automated rig as the second large force-grounded tactile resource of 2026.
