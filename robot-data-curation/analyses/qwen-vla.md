# Qwen-VLA — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Qwen-VLA: Unifying Vision-Language-Action Modeling across Tasks, Environments, and Robot Embodiments |
| **Org** | Alibaba Qwen |
| **Date** | 2026-05 |
| **Artifact** | arXiv 2605.30280 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full report with a published mixture table including sampling proportions** |
| **Backbone** | Qwen3.5-4B VLM + DiT flow-matching policy head |
| **Stance** | **Unify the *interface*, preserve the *semantics*.** Explicitly refuses to force all embodiments into one physical action space. |
| **Relationship** | Predecessor in the same stream as **[Qwen-RobotManip](../qwen-robotmanip/dataprocess.md)** (2026-06), which supersedes it with a far heavier cleaning cascade |

## 1. The published mixture — one of the few explicit tables in the field

| Data source | Proportion |
|---|---:|
| **Robot manipulation trajectories** (real + simulated) | **74.2%** |
| Navigation trajectories | 7.5% |
| Human egocentric trajectories | 6.0% |
| Synthetic simulation trajectories (theirs) | 3.7% |
| General vision-language data | 3.4% |
| Spatial grounding (2D) | 2.5% |
| Autonomous-driving VQA | 2.4% |
| Fine-grained embodied action caption | 0.2% |
| **Total** | **100.0%** |

Five data families: robot manipulation, human egocentric, synthetic simulation, navigation/trajectory-centric, and auxiliary vision-language.

Named sources include **RobotSet, Galaxea, AgiBot World, RoboCOIN, RoboMIND V1/V2, RDT-1B, DROID, BridgeData V2, RH20T, RT-1, BC-Z.**

Two things stand out against the rest of the survey: **human video is only 6%** here (vs 65% synthesized-from-human in Qwen-RobotManip and 100% in DYNA-2), and **navigation data is included at 7.5%**, which almost no other manipulation-focused corpus does.

## 2. The design decision — unify the tensor, not the semantics

> *"We **unify the tensor interface and masking scheme, but do not force all embodiments into a single physical action semantic space**. Each dataset preserves its **native control convention**, specified through the **embodiment prompt** and **dataset-specific normalization**."*

This is a materially different bet from ABot-M0 (convert everything to delta-EEF + axis-angle) and Qwen-RobotManip (canonical state-action vector + camera-frame delta pose). Qwen-VLA keeps conventions intact and *declares* them, similarly in spirit to π₀.₇'s `Control Mode: joint|ee` token but pushed much further.

### Embodiment-aware prompt conditioning
The prompt template carries the physical configuration explicitly:
```
[robot tag] [optional modifiers: waist, mobile base].
The control frequency is {FPS} Hz.
Please predict the next {chunk_size} control actions to execute the following task: {instruction}.
```
- Robot tag and modifiers set **per embodiment**
- **FPS and chunk_size reflect each dataset's *original* control frequency and prediction horizon** — no resampling to a common rate

Where ACE-Ego-0 solves control-frequency heterogeneity by **re-indexing chunks in physical time**, Qwen-VLA solves it by **telling the model the frequency**. Both are valid; the trade is normalization cost against model burden.

## 3. Loss-level handling of ragged action spaces

**Two-level averaging** in the action loss:
> *"This two-level averaging ensures that **each control dimension contributes equally to the gradient regardless of how many channels a given embodiment uses**, and that **padded positions are fully excluded**."*

A subtle but important correction: with naive summation, a 26-DoF humanoid would dominate gradients over a 7-DoF arm purely by channel count. This is a *data-balancing intervention implemented in the loss* rather than in the sampler — worth noting alongside ABot-M0's sampling-level balancing.

## 4. Mixture sampling
> *"Within each mini-batch we mix samples from all task families according to a **fixed sampling ratio**, so that every optimization step jointly updates the backbone and the action expert with manipulation, VLN trajectory, and vision-language signals."*

Combined loss `λ_act · L_act + λ_vl · L_vl`, with weights *"tuned to balance the gradient magnitudes of the two objectives"* — gradient-magnitude balancing rather than hand-set ratios.

## 5. Quality control reported in this stream
The stream's documented quality controls (also referenced in follow-up work) include filtering trajectories with **missing images, invalid states, or inconsistent action lengths**; discarding **projected gripper points falling outside the image or with negative depth**; removing **object boxes with invalid geometry or unstable category assignment**; and pruning **inconsistent chain-of-thought fields**. These are geometric-validity and schema-consistency checks rather than physics-based ones.

## 6. Unified output space
Manipulation actions, navigation waypoints, and human egocentric motions all live in *"a shared action-and-trajectory prediction space"*, with human motion in structured pose spaces (**MANO** or skeletal joint sequences). One inference interface across task families.

## 7. What they do not do
- No state–action causality check, no FK consistency check, no video–state rendering check (all introduced later in **Qwen-RobotManip**).
- No dedup.
- No published rejection rates or per-source hour counts.
- No fitted scaling law.

## 8. Transferable takeaways
1. **Publish the mixture proportions.** The 74.2/7.5/6.0/3.7/… table is the kind of disclosure that makes a corpus comparable at all.
2. **Declare conventions instead of converting them** — embodiment prompt + native FPS + native chunk size is a low-cost alternative to full harmonization.
3. **Balance gradients across action dimensionality**, or wide-DoF embodiments will silently dominate training.
4. **Balance objectives by gradient magnitude**, not by intuition.
5. Read against **Qwen-RobotManip**: within one month and one team, the strategy shifted from *declare heterogeneity* to *aggressively align and filter it* — and the later paper reports that alignment is what unlocked log-linear scaling. That trajectory is itself evidence about which approach scales.
