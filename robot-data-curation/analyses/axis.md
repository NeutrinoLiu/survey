# AXIS — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | AXIS: A Growable Community-Driven Data Engine for Scalable Robot Manipulation |
| **Date** | 2026-07 |
| **Artifact** | arXiv 2607.21588 (`paper.pdf`, `paper.html`); axisaiorg.github.io/AXIS-V1/ |
| **Disclosure level** | **A — full paper; the data engine *is* the contribution** |
| **Corpus** | **207 tasks, 50K+ trajectories**, browser-collected, versioned into snapshots |
| **Stance** | **Datasets should be *growable*, and growth must be *measurable*.** |

## 1. Thesis — the closed-and-centralized critique

> *"A fundamental limitation of existing manipulation data pipelines is that data collection remains largely **closed and centralized**. Expert operators or dedicated teams gather demonstrations on local hardware, process them offline, and release the resulting dataset as a fixed benchmark. While this paradigm offers strong control over data quality, it does not naturally support continual task expansion, broad community participation, or rapid iteration."*

Consequence: *"robot datasets have grown much more slowly than the models they are intended to train."*

The proposal: *"rather than serving as static collections of demonstrations, [datasets] should provide mechanisms for continuously generating, collecting, validating, processing, and evaluating new data. Such growth should also be **structured**, organizing new tasks and demonstrations into **reproducible dataset snapshots** that support consistent benchmarking over time."*

## 2. Three-layer architecture

### Infrastructure layer — removing the collection bottleneck
- **Automated task generation** from high-level instructions + object assets + scene layouts + **task-specific success conditions**. Success conditions are generated *with* the task — so every task is machine-verifiable from birth.
- **Browser-based MuJoCo-WASM teleoperation** — no specialized hardware, no local simulator install, commodity input devices. This is the mechanism that makes community contribution possible at all.
- New tasks are also **automatically validated** before deployment.

### Dataset layer — the cleaning pipeline
Raw community demonstrations → training-ready data via:

| Step | Purpose |
|---|---|
| **Unified trajectory representation** | Common schema across all contributors |
| **Automated success validation** | Replay against the task's own success checker — the payoff of generating success conditions with the task |
| **Quality filtering** | Reject sub-threshold demonstrations |
| **Static-segment removal** | Strip idle periods (cf. Hydra-0's static-window filter, HumanNet's quality filter) |
| **Trajectory smoothing** | Suppress jitter from commodity input devices — a defect specific to browser/mouse teleoperation |
| **Resampling** | Normalize control frequency across contributors |
| **IsaacSim-based visual and physics augmentation** | Cross-simulator augmentation |
| **Versioned task snapshotting** | Reproducible dataset states |

**Automated success checking is the crux.** Crowd-sourced collection is only viable if validation is free; AXIS gets that by co-generating the success condition with the task. Contrast every human-video corpus in this survey, where "did the demonstration succeed?" is unanswerable without a human.

### Model layer
Trains/evaluates both conventional visuomotor IL policies and modern VLAs using **shared task definitions and success checkers** — so the same predicate validates data and scores policies.

## 3. Making growth measurable — the snapshot protocol

> *"It organizes demonstrations into progressively larger task snapshots and evaluates policies using a fixed held-out protocol, which enables **controlled scaling studies in which the policy architecture, training budget, rollout protocol, evaluation tasks, and success criteria remain fixed while only the training snapshot grows**."*

This is the cleanest data-scaling experimental design in the survey: everything is pinned except corpus size. AXIS-25% / 50% / 100% snapshots are the rungs.

## 4. Scaling evidence

| Snapshot | Success rate |
|---|---|
| AXIS-25% | 84.7% |
| AXIS-50% | 85.7% |
| **AXIS-100%** | **88.8%** |

- Continual pretraining of **π₀.₅** on AXIS-100% improves overall **LIBERO-Plus by 5.8%**, and beats a **volume-matched RoboCasa365** continual-pretraining baseline by **37.3%** — a same-volume comparison, which isolates data *composition* from data *quantity*.
- Largest gains under perturbation: **sensor-noise +16.6%**, **camera +15.6%**, **robot-pose +5.1%**, **layout +3.1%**. The augmentation stages target exactly these axes, and the evaluation confirms the targeting.

## 5. What they do not do
- Simulation-only (MuJoCo/IsaacSim); no real-robot data in the engine, though real-world validation is reported.
- Quality-filter thresholds not published numerically.
- No dedup across contributors — plausible risk when many people demonstrate the same generated task.
- Scaling deltas between 25% and 50% are small (84.7 → 85.7), so the curve is not strongly resolved.

## 6. Transferable takeaways
1. **Generate the success checker with the task.** It converts validation from a human cost into a free automatic filter, and makes crowd-sourcing tractable.
2. **Version the corpus into snapshots** so scaling studies are reproducible and comparable over time.
3. **Pin everything but corpus size** when claiming a data-scaling result.
4. **Match your augmentation axes to your evaluation perturbation axes** — AXIS's biggest wins are exactly where it augments.
5. **Trajectory smoothing is mandatory for non-expert input devices** — a defect class that professional teleop corpora never see and therefore never filter for.
6. **Volume-matched baselines** (AXIS-100% vs RoboCasa365 at equal volume) are the right way to argue that your data is better rather than merely larger.
