# ACE-Ego-0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | ACE-Ego-0: Unifying Egocentric Human and Robotic Data for VLA Pretraining |
| **Org** | ACE (see also [ACE-Brain-0.5](../ace-brain-0.5/dataprocess.md), the stream's latest) |
| **Date** | 2026-06 |
| **Artifact** | arXiv 2606.17200 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A+ — publishes a complete numeric hyperparameter table for every pipeline stage.** Among the most reproducible pipelines in this survey. |
| **Corpus** | **6.0K+ h** = **1.48K h** pseudo-action-labeled human video (from 6 ego datasets) + **4.53K+ h** sensor-logged multi-embodiment robot + simulation |
| **Stance** | **Supervision fidelity is a first-class variable.** Don't just align representations — *weight the loss by how much you trust each label.* |

## 1. Thesis — alignment is necessary but not sufficient

> *"As supervision quality varies substantially across data sources, **representation alignment alone is insufficient** to achieve effective mixed-source pretraining."*

And the failure mode of ignoring it:
> *"equivalent treatment forces the policy to **directly mimic the artifacts and failures of the reconstruction pipeline**."*

This is the sharpest statement in the survey of the risk in pseudo-labeled human video: you don't just learn the human's behavior, you learn your hand-tracker's bugs.

## 2. Three axes of heterogeneity, addressed jointly

ACE-Ego-0's critique of prior work is that each axis has been solved in isolation: *"a shared action vector does not guarantee aligned coordinate frames; fixed-length action chunks span disparate physical durations under varying control frequencies; and kinematic structures are often implicitly absorbed via simple dataset IDs."*

| Axis | Mechanism |
|---|---|
| **Spatial** | **Unified camera-space action representation** — no coordinate transformations beyond a standard camera extrinsic |
| **Structural** | **Cross-embodiment morphology conditioning** — embed robot kinematic descriptions; **learned surrogate embeddings for human-video sources** |
| **Temporal** | **Time-aligned action chunking** — index future actions by **physical timestamps rather than frame indices**, ensuring consistency across datasets at different control frequencies |

Time-aligned chunking is an underrated fix: a fixed 16-frame chunk means 0.53 s at 30 Hz and 3.2 s at 5 Hz. Every corpus mixing OXE subsets has this problem; few address it explicitly.

## 3. The five-stage egocentric pipeline — with published thresholds

### Stage 1 — Dataset curation
Sources selected on three criteria: **egocentric viewpoint**, **diverse real-world interaction scenes**, **high-quality action-centric captions** → 6 datasets. Standardized into a unified storage format with consistent metadata: clip identifiers, frame indices, camera intrinsics (when available), narrations, **and licensing information**.
- **Discard clips < 4 s or > 30 s** — *"unlikely to contain complete manipulation primitives at the downstream temporal granularity."*

### Stage 2 — Video selection (cheap filters before expensive reconstruction)
Explicitly ordered for cost: *"Before applying computationally intensive geometric reconstruction, we adopt an ego-interaction filter."*
- **Face-detection filter**: *"strong face detections serve as an effective signal of non-egocentric or observer-centric viewpoints, which rarely contain usable manipulation trajectories."* Discard if max face-detection confidence **> 0.5**. A clever, cheap proxy for "is this actually first-person?"
- **Caption-based filter**: retain only clips whose narrations contain **at least one manipulation verb AND one manipulable object noun**.

### Stage 3 — 3D hand reconstruction (three sub-stages)
1. **2D tracking** — SAM3-based tracker for temporally consistent hand boxes and masks. **Discard detections with keypoint confidence < τ_kp = 0.4 or track length < ℓ_min = 15 frames.**
2. **Local pose estimation** — HaMeR reconstructs MANO `{β, θ_t, t_local}`.
3. **Global trajectory optimization** — two stages, because *"per-frame reconstruction suffers from depth ambiguity, occlusions, and temporal jitter"*:
   - Stage A: **N_root = 30** iterations for globally consistent root translation/orientation
   - Stage B: **N_smooth = 200** L-BFGS iterations minimizing reprojection error + temporal smoothness
     `L_smooth = L_reproj + λ_tv Σ_t ‖t_{t+1}^global − 2t_t^global + t_{t−1}^global‖²`, with **λ_tv = 1.0**
   - Per-frame camera poses from **VIPE**

Important detail: *"The optimized global trajectory is used **only for temporal consistency**; the final pseudo-action labels are transformed back into the head-camera frame before training."* Global optimization for smoothing, camera frame for learning.

### Stage 4 — Action parameterization
- Parameterization: **wrist origin, palm-plane orientation, thumb-to-palm gripper proxy.**
- **Storage layout**: 16-D bimanual on disk = per hand (3 position + 3 XYZ Euler + 1 gripper + **1 activity flag**) × 2. At training, Euler → continuous 6D rotation → **22-D action vector**.
- **Gripper normalization**: thumb-to-palm distance linearly normalized to robot stroke range **[0.04, 0.10] m**.
- **Degeneracy detection**: trajectories whose **10th–90th percentile range `d90 − d10 < τ_grip = 1.5 cm`** are treated as degenerate (*"e.g., closed-fist motion with no grasp transition"*) and assigned a **constant neutral gripper state** — corrected, not discarded.

### Stage 5 — Quality control: four filters
> *"removes corrupted or behaviorally implausible human episodes before they are collected into the mixed-source pretraining datasets."*

| Filter | Rule | Threshold |
|---|---|---|
| **Completeness** | No NaN/Inf; contiguous frame indices; quaternion normalization `\|‖q‖ − 1\| ≤ τ_quat` | τ_quat = **1e-3** |
| **Static** | Discard if neither hand exceeds per-second motion energy τ_static | source-specific |
| **Spike** | Reject if inter-frame positional change exceeds **κ_spike = 3σ** of the per-episode velocity distribution on **more than ρ_spike = 5%** of frames — *"typically indicates tracking failures or reconstruction artifacts"* | 3σ / 5% |
| **Bimanual** | Remove episodes with implausible dual-arm behavior based on **anomalous inter-hand distance statistics or weak temporal correlation between the two hands** | source-specific |

> *"We record the corresponding thresholds in the released data manifests since they depend on source-level hand-detection density."* — thresholds shipped *with the data*, which is exactly right.

The **spike filter is per-episode-relative** (3σ of that episode's own velocity distribution), so it adapts to fast and slow episodes without a global constant. The **bimanual filter** is unique in this survey: checking whether two hands behave like two hands is a physically-motivated plausibility test no other pipeline runs.

## 4. Reliability-aware training objective — the distinctive contribution

Rather than a binary keep/drop, supervision is **graded**:

- **Sensor-logged robot trajectories** supervise the **primary flow-matching objective**.
- **Pseudo-actions from human video** are:
  - **down-weighted**,
  - used as **auxiliary supervision**,
  - **restricted primarily to noiseless position channels** (rotation and gripper channels, where reconstruction is least reliable, contribute less), and
  - **modulated by both dataset-level and step-level quality estimates**.

> *"we ensure that high-fidelity robot data anchor the primary action expert, while human videos provide safe, robust, and complementary auxiliary supervision."*

This is the most refined answer in the survey to the mixed-fidelity problem — a **continuous trust weight per source and per timestep**, sitting between Dyna-2's binary two-tier routing and π₀.₇'s explicit quality-token conditioning.

## 5. Deployment detail worth noting
Because actions are predicted in the head-camera frame, deployment needs only a standard extrinsic inverse:
`p̂_s = R_{cam←s}ᵀ (p̂_cam − t_{cam←s})`, with the 6D rotation reconstructed via Gram–Schmidt before the transform. Camera-space representation costs nothing at execution time.

## 6. Results & scaling evidence
- **RoboCasa GR1 TableTop: 72.8%** average success (vs GR00T-N1.6, Qwen3PI, FLARE, ABot-M0, JoyAI-RA, DIAL)
- **RoboTwin 2.0: 91.12% Easy / 90.62% Hard** (vs π₀.₅, Motus, LingBot-VLA, ABot-M0, JoyAI-RA, Hy-VLA)
- Real ARX bimanual platform: strong long-horizon contact-rich performance vs fine-tuned π₀.₅ and GR00T-N1.7
- **Ablations confirm each of morphology conditioning, time-aligned action chunking, and reliability-aware human supervision contributes**, and *"scaling pseudo-action-labeled human video on top of robot data yields further gains."*

## 7. What they do not do
- No dedup across the six ego sources (Ego4D/EPIC overlap is a known issue).
- No visual domain adaptation (no robot rendering into human video — contrast Qwen-RobotManip's H2R).
- Corpus is modest (6.0K h) relative to EgoScale/DYNA-2; the reliability machinery is validated at that scale only.
- Some thresholds (τ_static, bimanual) are source-specific and deferred to the manifests.

## 8. Transferable takeaways
1. **Order filters by cost.** Face detection and caption keyword checks before HaMeR + trajectory optimization saves the bulk of the compute.
2. **Face detection as an egocentricity test** — cheap, and catches observer-viewpoint contamination that no motion filter would.
3. **Per-episode-relative spike detection (3σ, >5% of frames)** adapts to episode dynamics automatically.
4. **Check bimanual plausibility** — inter-hand distance statistics and temporal correlation catch reconstruction failures invisible to per-hand checks.
5. **Correct degenerate grippers rather than dropping the episode** (closed-fist → neutral constant).
6. **Grade supervision, don't gate it.** Channel-restricted, down-weighted, quality-modulated auxiliary loss is strictly more expressive than keep/drop.
7. **Index action chunks by physical time, not frames**, whenever control frequencies differ across sources.
8. **Ship thresholds in the data manifest** so consumers can reproduce or re-tune the filtering.
