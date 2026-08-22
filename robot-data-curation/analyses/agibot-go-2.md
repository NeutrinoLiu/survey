# AGIBOT GO-2 + AGIBOT WORLD 2026 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **AGIBOT WORLD 2026** dataset (2026-04-07) · **GO-2** ViLLA embodied foundation model (2026-04-17 / 2026-08) |
| **Org** | AGIBOT (智元机器人), Shanghai |
| **Artifacts** | `page.html` (AGIBOT press release), `world2026.html` (The Robot Report coverage) |
| **Disclosure level** | ⚠️ **C — press release + trade-press coverage only.** No technical report, no numeric thresholds, no rejection rates. "Industrial-grade data-processing system" is asserted, never specified. Claims below are vendor claims. |
| **Corpus** | Phase 1: *"hundreds of hours"* of real-world data, commercial/service environments; **five-phase release plan** |
| **Predecessor** | AgiBot World Colosseo / GO-1 (arXiv 2503.06669, 2025) — the stream's peer-reviewed entry |
| **Stance** | **Free-form collection over scripted demonstration**, with hierarchical annotation and *retained failure data*. |

## 1. Collection strategy — free-form over scripted

> *"Unlike conventional datasets built on repetitive and scripted demonstrations, [AGIBOT] has taken a **free-form data-collection approach, in which teleoperators dynamically perform tasks based on real-time conditions**."*

Claimed benefit: *"significantly enhance diversity within each episode and improve generalization across multiple dimensions, including **object categories, initial configurations, and task execution sequences**."*

This is a curation decision made at the *protocol* level — the opposite of EgoVerse's standardized flagship-task protocol. EgoVerse constrains the task to make diversity measurable; AGIBOT unconstrains the execution to maximize within-episode variance. Both are defensible; they optimize different things (comparability vs. coverage).

Environments: commercial spaces, homes, everyday scenarios — *"the complexity, variability, and unpredictability that robots must handle in practice."*

## 2. The "does the data reflect the robot as a system?" framing

The stated design question is unusually well-posed:
> *"A fundamental question in embodied AI remains: **Does the data truly reflect how a robot operates as an integrated system?**"*

Three answers, each a data-collection intervention rather than a filter:

| Feature | What it changes about the data |
|---|---|
| **Whole-body control (WBC)** | Coordinated arms + waist + hands, so recorded trajectories are *"fluid… as a unified system rather than through isolated motions"* — avoids the artifact of arm-only data from a robot that actually has a torso |
| **First-person beyond-visual-range teleoperation** | *"The robot's perception is aligned with that of the operator"* — removes the observation mismatch between what the demonstrator saw and what the policy will see |
| **Force-controlled data collection** | *"capturing not only motion trajectories, but also real physical interactions"* — contact dynamics and force feedback recorded, not just kinematics |

The second point is the most important and most often neglected: if the teleoperator sees a third-person view or a different camera set than the policy will, every demonstration encodes information the policy cannot access. Aligning operator and robot perception is a *data-validity* fix, not an ergonomics one.

## 3. Sensing and the "industrial-grade" pipeline

Collected on the **G2** platform (high-performance joint actuators, multi-modal sensors, high-performance domain controller), with **Zhixing 90D grippers** and the dexterous **OmniHand**.

Synchronized multi-modal capture *"within a unified pipeline"*:
**RGB(D) · tactile signals · lidar point clouds · IMU · full-body joint states**

> *"each data episode undergoes **rigorous cleaning and validation through its 'industrial-grade' data-processing system**, ensuring readiness for large-scale model training."*

⚠️ This is the entire published description of the cleaning pipeline. No stages, thresholds, rejection rates, or failure taxonomy are disclosed. Treat as an assertion of capability, not evidence of one.

Also released: **1:1 digital twin environments in simulation**, with corresponding simulation data published alongside the real-world dataset — real/sim pairing at the scene level.

## 4. Hierarchical annotation framework

Phase 1 (imitation learning) combines four annotation layers:

| Layer | Content |
|---|---|
| **Task-level descriptions** | Segment-level instructions |
| **Action sequences** | Step-by-step execution |
| **Atomic skill labels** | e.g. *pull*, *place* |
| **Object annotations** | 2D bounding boxes + attributes (name, colour) |

## 5. The most notable curation decision — keeping failures

> *"**Importantly, error-recovery trajectories are also retained and annotated.** This hierarchical annotation framework—spanning from high-level tasks to low-level actions—provides the fidelity and **corrective priors** needed to train more robust and adaptive embodied agents."*

Deliberately retaining and *labelling* error-recovery trajectories puts AGIBOT alongside **π₀.₇** (which annotates mistakes and quality 1–5 and conditions on them) and against the default assumption that a demonstration corpus should contain only successes. Recovery behaviour cannot be learned from a corpus that filters out every episode in which something went wrong.

## 6. Release plan
**Five phases**, each aligned with a core research direction. Phase 1 = imitation learning, hundreds of hours, commercial and service environments.

## 7. GO-2 model
ViLLA embodied foundation model bridging planning and execution via **Action Chain-of-Thought**: generating a macro-plan — *"a sequence of 'action intents' that serve as a mental simulation of the task"* — decomposing instructions into ordered logical stages, to close what AGIBOT calls the **"Semantic-Actuation Gap."** Reported to bring together *"tens of thousands of hours of interaction data."*

The Action-CoT supervision requirement is what motivates the hierarchical annotation above: you cannot train a model to emit action intents without a corpus annotated at the intent level.

## 8. What is not disclosed
- No cleaning stages, thresholds, rejection rates, or defect taxonomy.
- No total corpus hours for GO-2's training set (only *"tens of thousands of hours"*).
- No mixture proportions, no scaling curves, no ablations.
- No independent verification of any claim.

## 9. Transferable takeaways
1. **Align operator perception with robot perception.** Demonstrations recorded under a different observation model than deployment encode inaccessible information.
2. **Record forces, not just kinematics** — contact dynamics are unrecoverable after the fact.
3. **Collect with the whole body if the robot has one**, or the corpus teaches a decomposition the hardware doesn't have.
4. **Retain and annotate error-recovery trajectories.** Failures are the only source of corrective priors.
5. **Free-form execution as a diversity mechanism** — a collection-protocol alternative to post-hoc diversity balancing.
6. **Publish 1:1 digital twins alongside real data** to make sim-real comparisons possible at the scene level.
