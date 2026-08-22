# ABot-M0 (UniACT-dataset) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | ABot-M0: VLA Foundation Model for Robotic Manipulation with Action Manifold Learning |
| **Org** | Alibaba AMAP |
| **Date** | 2026-02 |
| **Artifact** | arXiv 2602.11236 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full report; data cleaning is §2, and the full data-processing codebase is promised** |
| **Corpus** | **UniACT-dataset: >6M trajectories, 9,500+ hours, 20+ embodiments**, from **>7M source trajectories** across 6 public datasets |
| **Stance** | *Aggregation as an alternative to proprietary collection.* **"scale sets the floor, quality defines the ceiling, and diversity determines the scope."** |

## 1. Thesis

> *"given the high cost and hardware dependence of robotic data collection, can we integrate global open-source datasets, currently isolated in silos, into a unified foundation for training VLA models? … embodied intelligence need not emerge from closed, proprietary systems, but can instead develop through the aggregation of heterogeneous data."*

The diagnosis of why naive aggregation fails is the crispest in the survey:

> *"Action representations, coordinate systems, and control frequencies differ across datasets… The resulting fragmentation introduces noise and weakens learning, as **models must allocate capacity to memorize idiosyncrasies rather than acquire transferable [structure]**."*

## 2. Source taxonomy — three axes, no dataset has all three

| Axis | Exemplars | Role in the mixture |
|---|---|---|
| **Large scale** | OXE, OXE-AugE | Foundational pretraining corpus, broad scene/task coverage |
| **High quality** | AgiBot-Beta, Galaxea | Structured task design, fine-grained annotation, high physical fidelity — for capability refinement |
| **Embodiment diversity** | RoboCOIN, RoboMIND | Multiple morphologies/kinematics on unified platforms with careful alignment |

> *"It is difficult for one dataset to simultaneously satisfy all three characteristics"* — hence **dataset-specific curation strategies tailored to their structural and semantic properties**, rather than one uniform pipeline.

Explicit per-source policy:
- **OXE** → foundation for single-arm coverage
- **OXE-AugE** → augments embodiment/morphological variation within single-arm
- **AgiBot-Beta, Galaxea** → high-quality observations and temporally coherent actions; but **AgiBot-Beta's sampling ratio is deliberately reduced** because *"despite its substantial scale, [it] features only a single embodiment type"* — an explicit anti-bias intervention
- **RoboCOIN, RoboMIND** → prioritized for their fine-grained long-horizon task decompositions across morphologies

## 3. Defect taxonomy — what they actually found in public data

An unusually candid enumeration, useful as a checklist for anyone consuming these corpora:

| Defect | Detail |
|---|---|
| **Format fragmentation** | Raw sources span LeRobot v2, LeRobot v3, RLDS, and dataset-native formats |
| **Non-English task prompts** | French, Spanish, Chinese found in supposedly English corpora |
| **Garbled text** | Nonsensical character sequences |
| **Redundant sentences / empty content** | *"severely impair intent parsing and language-action alignment"* |
| **Split annotations for long-horizon tasks** | Task files hold only high-level scene descriptions; concrete subtask instructions live in *separate files* and must be joined to decompose trajectories |
| **Frame-rate discrepancies** | *"dropping as low as 5 FPS in some [OXE] subsets—leading to ambiguous time steps"* |
| **Unspecified action semantics** | Whether a dimension encodes translation, rotation (and in which representation), or gripper control *"is often unspecified"* |
| **Incomplete action vectors** | *"some action vectors lack critical components altogether"* |

And the consequence they name explicitly:
> *"neglecting proper text alignment will cause the trained model to effectively **degenerate into a vision-action (VA) model lacking proper language grounding**."*

## 4. Cleaning pipeline

- **Format unification**: all trajectories converted to **LeRobot v2** as the common standard.
- **Instruction filtering**: remove episodes with **empty task fields**; filter **garbled instructions** with nonsensical sequences; **normalize** the rest.
- **Subtask decomposition**: join separate subtask files to split long-horizon trajectories into meaningful units.
- Multi-stage filtering to remove typical-issue samples and refine the remainder.

**Attrition: >7M source trajectories → >6M retained (~15% removed).**

## 5. Standardization — three design principles

**(a) Delta actions in EEF space with rotation vectors.**
- Absolute → **delta** actions (*"simplifies and improves the efficiency of model training"*).
- **EEF rather than joint space** to bridge the embodiment gap.
- All orientation representations (Euler, rotation matrix, quaternion) → **rotation vectors (axis-angle)** `r = θk`, `r ∈ R³`, `θ ∈ [0,π]`, `‖k‖=1`, chosen explicitly *"to avoid singularities"* and for *"greater stability in action prediction, particularly for fine-grained rotational manipulation."*
- Output: two 7-D vectors (left/right arm), each `[Δx, Δy, Δz, r, gripper]`.

**(b) Pad-to-dual-arm.** Single-arm trajectories zero-pad the unused arm and are *"uniformly treated as right-arm executions within a dual-arm configuration."* The model always emits dual-arm sequences but activates only relevant channels. Lets one network learn *when* to use one arm vs. bimanual coordination.

**(c) Distribution accounting.** **OXE-AugE alone is 67% of total volume**; OXE second; the four dual-arm datasets together only **~17.2%**. Publishing this imbalance is what motivates the sampling study below.

## 6. Sampling strategy — the distinctive contribution

Three strategies compared, with **inequality metrics** applied to the resulting effective training distribution:

| Strategy | Effect | LIBERO-Plus |
|---|---|---|
| **Trajectory-uniform** | *"largely preserves the original dataset scale imbalance, causing training to be dominated by AgiBot-G1 and resulting in a highly concentrated embodiment distribution, which increases the risk of homogenization"* | 71.3 |
| **Embodiment-uniform** | — | 71.6 |
| **Task-uniform** | *"substantially alleviates this issue by increasing the sampling visibility of multi-task but single-embodiment data from RoboCOIN… allows long-tail embodiments to receive more exposure while still retaining AgiBot-Beta as the primary data source"* | **72.4** |

**Skill-level analysis via Lorenz curve and Gini coefficient:** all strategies show long-tailed skill distributions, but task-uniform produces *"a noticeably less concentrated probability mass… a Lorenz curve closer to the equality line and a lower corresponding Gini coefficient."*

Using **Gini/Lorenz to quantify corpus concentration** is a genuinely portable idea — it turns "is our mixture balanced?" from a judgment call into a number.

## 7. Results
98.6% (LIBERO), 80.5% (LIBERO-Plus), 58.3% (RoboCasa GR1 Tabletop), 81.2% (RoboTwin 2.0), beating π₀.₅, UniVLA, OpenVLA-OFT — *"demonstrating that high-performance, generalizable embodied intelligence can be achieved through systematic engineering **without reliance on proprietary data**."*

## 8. What they do not do
- No signal-level physics checks (no FK consistency, no state–action causality test — contrast Qwen-RobotManip).
- No dedup across overlapping public corpora (OXE and OXE-AugE overlap substantially by construction).
- No video-quality filtering.
- Sampling-strategy gains are modest (71.3 → 72.4), so the balancing argument rests more on distributional analysis than on downstream deltas.

## 9. Transferable takeaways
1. **Publish the defect taxonomy** you find in public corpora — non-English prompts, 5 FPS subsets, and unspecified action semantics are facts every downstream consumer needs.
2. **Per-source curation strategy, not a uniform pipeline.** Each corpus contributes a different axis and should be filtered and weighted for that axis.
3. **Down-weight large single-embodiment sources explicitly** to counteract homogenization.
4. **Axis-angle over Euler/quaternion** for cross-dataset rotation harmonization.
5. **Quantify mixture balance with Gini/Lorenz**, and choose the sampling granularity (trajectory / embodiment / **task**) that minimizes concentration.
