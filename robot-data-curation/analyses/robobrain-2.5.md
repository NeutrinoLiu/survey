# RoboBrain 2.5 (BAAI) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | RoboBrain 2.5: Depth in Sight, Time in Mind |
| **Org** | Beijing Academy of Artificial Intelligence (BAAI) |
| **Date** | 2026-01 |
| **Artifact** | arXiv 2601.14352 (`paper.pdf`, `paper.html`) |
| **Predecessors** | RoboBrain (2502.21257) → RoboBrain 2.0 (2507.02029) → **RoboBrain 2.5**; sibling: *Wujie RoboBrain Orca* (2026) |
| **Disclosure level** | **A — full technical report with a dedicated §3 Training Data** |
| **Stance** | **Choose the label representation that maximizes data reusability.** |

## 1. The (u, v, d) decision — a curation choice disguised as a representation choice

RoboBrain 2.5's most transferable idea is about **label format**, and the justification is explicitly about data:

> *"Rather than predicting 3D coordinates in the form (x, y, z) in camera or world frame, we adopt a **decoupled (u, v, d) representation**, which can be trivially projected to 3D coordinates using known camera intrinsics… this formulation **not only promotes data reusability but also ensures compatibility with existing 2D datasets**, thereby boosting multi-task learning performance through **co-training across complementary tasks and modalities**."*

Predicting `(x, y, z)` locks you out of every 2D-annotated corpus in existence. Predicting `(u, v, d)` — image coordinates plus depth — makes the entire 2D grounding/detection literature immediately usable as training data, with the 3D lift deferred to a known camera intrinsic.

This is the same family of move as Hydra-0's image-plane action flow and ACE-Ego-0/Qwen-RobotManip's camera-space actions: **stay in the observation frame, because that is where the data lives.** RoboBrain 2.5 states the data-reuse rationale most explicitly.

## 2. Training data composition
§3 organizes the corpus around three capability targets — *"spatial understanding, temporal modeling and causal reasoning in embodied settings"* — with named sub-corpora including **General MLLM Data** (§3.1) and **Spatial Reasoning Data** (§3.2). RoboBrain 2.0's lineage established the pattern: a progressive three-stage curriculum of foundational spatiotemporal learning → embodied enhancement → chain-of-thought reasoning, each stage with its own mixture.

## 3. The value/progress data pipeline — three stages with human keyframes

RoboBrain 2.5 formulates value estimation as **task progress**, making the model *"a vision-language estimator designed to infer fine-grained, real-time progress from visual inputs."* To build that supervision:

> *"To guarantee generalizability across diverse embodiments and task families, we implement a **three-stage data curation pipeline** handling diverse data origins. This process spans from **raw video segmentation** to a systematic, **hop-based labeling strategy**."*

**Step-wise task-progress discretization:**
- Raw multi-view video trajectories are segmented into sub-tasks using **human-annotated multi-view keyframes** `{K₀, K₁, …, K_N}`, where `K₀` is the initial observation, `K_N` the final success observation, and each `K_j` a set of **synchronized multi-view keyframes**.
- *"To obtain dense supervision, we perform **adaptive sampling within each segment**."*

Two things worth noting:
1. **Keyframes are the human-annotation budget, and everything else is derived.** Annotating N boundary frames per trajectory and adaptively sampling between them converts a small labeling effort into dense supervision — the same economics as PhysBrain's record→QA compilation and π₀.₇'s episode-level metadata.
2. **Multi-view synchronization is enforced at the keyframe level**, which is what makes the segmentation consistent across cameras.

Compare **π*₀.₆/RECAP**, which obtains the same progress signal *without* human keyframes by fitting a value function to returns. RoboBrain 2.5 buys accuracy with annotation; RECAP buys scale with automation. Both target the identical quantity.

## 4. Position in the survey
RoboBrain sits in the **structured-reasoning-supervision** wing alongside PhysBrain 1.0 and Tencent's RxBrain: human video and robot trajectories are converted into QA, spatial reasoning, and progress-estimation supervision rather than into action labels. The three works independently converge on the pattern of **taxonomy → structured records → generated supervision**.

## 5. What they do not do
- No filtering cascade, rejection rates, or defect taxonomy published.
- Human keyframe annotation is a scaling bottleneck (the cost RECAP avoids).
- No dedup; no per-source hour accounting.
- No fitted data-scaling curve.

## 6. Transferable takeaways
1. **Pick the label space that maximizes corpus compatibility.** `(u, v, d)` unlocks every 2D dataset; `(x, y, z)` does not.
2. **Annotate boundaries, derive the interior.** Human keyframes + adaptive sampling turns O(N) labels into dense supervision.
3. **Synchronize annotation across views at the keyframe level**, so multi-view segmentation stays consistent.
4. **Formulate value as task progress** — it yields a per-timestep quality/progress signal usable for both supervision and filtering.
