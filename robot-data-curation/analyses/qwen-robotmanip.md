# Qwen-RobotManip — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Qwen-RobotManip Technical Report: Alignment Unlocks Scale for Robotic Manipulation Foundation Models |
| **Org** | Alibaba Qwen Team |
| **Date** | 2026-06-16 (v2 2026-06-17) |
| **Artifact** | arXiv 2606.17846 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full technical report**, stage-by-stage pipeline with named algorithms, thresholds, and a reported rejection rate |
| **Corpus** | **~38,100 h** manipulation + ~28M VL samples |
| **Stance** | *Alignment is the prerequisite for scaling* — the corpus only scales because representation is harmonized first |

## 1. Sources & scale

| Stream | Embodiment | Sources | Hours |
|---|---|---|---|
| Robot | Single-arm | OXE (Fractal/Bridge/BC-Z ~600 h), DROID (~95K traj, 500 h), RH20T (~1,100 h) | 3,808 |
| Robot | Dual-arm | AgiBotWorld-Beta (~2,400 h, gripper subset, ~200 tasks), RoboMIND + RoboMIND 2.0 (~1,400 h), RoboCOIN (~430 h, 10 embodiments), RDT-1B (29 h) | 6,744 |
| Robot | Mobile & humanoid | InternData-A1 (sim, >3,600 h), Galaxea Open-World (~500 h) | 868 |
| Human | Hands | EgoDex (732/829 h used), VITRA (Ego4D + EPIC-KITCHENS subsets, ~247 h), EgoVerse (industry portion, ~954 h) | 1,933 |
| Synthetic | 15 dual-arm platforms | Human-to-Robot (H2R) rendering of the 1,933 h ego pool | **24,808** |

Headline claim: **zero proprietary collection.** 65% of the corpus by time is synthesized, not collected.

## 2. Ingestion & normalization

- **Canonical state–action vector** with **per-dimension binary masking** — one fixed template absorbs every morphology; absent DoFs are masked rather than zero-padded silently.
- **Camera-frame delta-pose parameterization** — end-effector actions expressed relative to the camera so visually similar motions are numerically proximate regardless of each dataset's base frame.
- **Hand pose unification** — all ego sources converted to MANO parameters + 21 keypoints; sources without native MANO get parameters recovered by optimization-based fitting.
- **Quantile normalization** `[q01, q99] → [-1, 1]`, computed per embodiment type.

## 3. Filtering & quality control — the five-stage signal pipeline

Applied to **all** datasets before training. Thresholds are set *per dataset* by embodiment type, rotation representation, real-vs-sim provenance, and base mobility.

| Stage | Method | Action on failure |
|---|---|---|
| **1. Sudden-change detection** | Cascaded median filter + Savitzky–Golay smoothing to extract trend; then three deviation signals — absolute residual, 2nd-order difference (acceleration), 3rd-order difference (jerk). Frame flagged when residual exceeds a scaled threshold **AND** acceleration or jerk also exceeds its threshold (the conjunction suppresses false positives from slow drift while staying sensitive to transients). | Frame removal → full episode discard, per dataset. In InternData-A1 sudden changes come exclusively from collisions, so the whole episode is dropped. |
| **2. State–action trend alignment** | Enforces a causal invariant: commands must lead or coincide with the state change they cause. Smooth both streams, estimate optimal temporal lag by cross-correlation, compute a **directional agreement (DA)** metric on lag-aligned first differences. Delta-action datasets are integrated to absolute values first. | Dimensions with DA below a dataset-specific threshold (**typically 0.6–0.7**) flagged, episodes excluded. **Reported hit rate: 81% of RoboMIND UR-type episodes failed and were excluded.** |
| **3. Extreme-value filtering** | Per-dimension q1/q99 per embodiment; drop frames outside `[q1 − α(q99−q1), q99 + α(q99−q1)]`. Protects the quantile normalizer from distortion. | Frame removal. **Gripper dims exempt** — bimodal by nature. |
| **4. Joint↔EEF forward-kinematics consistency** | FK via Pinocchio from each URDF, compared against logged EEF poses. Diagnoses sign-convention flips, differing EEF frame definitions, wrong rotation representations, wrong base-frame assumptions, erroneous logging. | **Correction, not deletion** — TCP redefinition absorbs constant offsets; shoulder-relative bimanual poses transformed to world frame. |
| **5. Base-frame / EEF orientation alignment** | Per-dataset rotation corrections so +x is always robot-forward. | Correction. |

### Cross-modal consistency checks (beyond signals)

- **Check 1 — Instruction consistency (3-stage VLM pipeline).** (a) *Temporal normalization*: long episodes decomposed into subtask-level segments so each clip is one action unit. (b) *Structured reasoning-guided annotation*: the VLM is forced to reason about manipulated objects, action semantics, temporal ordering, and agent–environment interaction **before** emitting a binary label — explicitly to suppress heuristic answers. (c) *Multi-expert cross-model adjudication*: clips flagged non-aligned/ambiguous are re-judged by **multiple independent VLMs with cross-model voting**. Inconsistent samples excluded.
- **Check 2 — Video–state consistency.** Render the robot into the image plane from URDF + logged joint states; segment the *actual* robot mask with a **fine-tuned SAM3**; measure mask overlap. Low overlap → filtered. This catches calibration drift and mislabeled embodiments that signal-level checks cannot.
- **Check 3 — Video quality.** Remove black, corrupted, blurred, and prolonged-static frames, using image checks *jointly with state/action signals* to detect redundant static periods (typically at episode boundaries). **Task-critical keyframes such as gripper-closure events are explicitly preserved** so visually subtle but semantically decisive transitions survive.

## 4. Deduplication
Not described as a distinct stage. Diversity is managed by source selection and subset retention (e.g. only 3 of OXE's subsets retained) rather than embedding-space dedup.

## 5. Annotation
Language annotations are *verified*, not generated from scratch, by the Check-1 VLM cascade. Subtask decomposition supplies the temporal units. A separate **~28M-sample VL co-training mixture** (RoboPoint, RefSpatial, PixMo, CapsFusion, proprietary, plus synthesized embodied chain-of-thought and egocentric video-understanding data) runs as a second stream to stop the VLM backbone eroding under action-prediction pressure.

## 6. Action-space harmonization / retargeting

Human→gripper retargeting, fully specified:
- Virtual finger `k_vf = 0.7·k_index + 0.3·k_middle`
- EEF position `p = ½(k_thumb + k_vf)`; gripper width `w = ‖k_thumb − k_vf‖₂`
- Orientation frame: grasp axis `z` along the jaw line, `y` = normal of the plane spanned by `z` and wrist→fingertip `d`, `x = y × z`

**Action-speed alignment** — a distribution-matching step rarely made explicit elsewhere: human hands move much faster than teleoperated robots, so each ego source is frame-subsampled to match robot speed. EgoDex → 60% (~1.7× slower), EgoVerse → 45% (~2.2× slower), VITRA → 25% (~4× slower).

## 7. Augmentation & synthesis — the H2R engine (6 steps)

`Input ego video → ② retarget + smooth → ③ arm segmentation (SAM3) → ④ hand removal (ProPainter inpainting) → ⑤ base-pose search + MuJoCo IK → ⑥ depth-guided compositing`

Rendered across **15 robot morphologies** (Panda, UR5e, ARX-L5, xArm7, Sawyer, Kinova Gen3, IIWA, Jaco, FR3, UR10e, ViperX, WidowX, Piper, YAM, AgileX ALOHA), turning 1,933 h into 24,808 h — a **12.8× multiplier**. Base-pose search + IK is the reachability filter: a human trajectory that no valid base placement can realize is rejected for that morphology.

## 8. Mixture weighting
Fixed **7:3 robot-to-auxiliary ratio** in the reported ablations. Dual-stream co-training over manipulation data and the VL stream.

## 9. Scaling evidence

The paper's central empirical claim is that **data scaling is contingent on representation alignment**:
- With the unified EEF representation, best validation MSE **decreases approximately log-linearly with training data volume across the full range**.
- The ablated variant (raw per-embodiment action fields zero-padded to 80 dims, no cross-embodiment alignment) **fails to exhibit scaling behavior at all**.
- Quote: *"alignment is what converts additional data volume into improved capability."*

H2R ablation at fixed 7:3 ratio (robot-only / +raw ego / +H2R): RoboTwin-Clean2Rand Hard 54.7 → 55.0 → **58.7**; LIBERO-Plus avg 87.1 → **89.0**, with the Camera axis gaining +7.2 (72.8 → 80.0). Raw ego data barely helps; *synthesized* ego data does.

## 10. Stated limitations / what they do not do
- No embedding-based dedup or semantic-diversity balancing.
- No proprietary data — deliberately, but it caps embodiment coverage to what open datasets contain.
- Authors argue standard in-domain benchmarks **systematically fail to measure pretraining quality**, and build OOD benchmarks (RoboTwin-IF, RoboTwin-XE) instead — a data-evaluation, not just data-cleaning, position.
- Context-conditioned variant hesitates at episode start (zero-padded history); both variants released.

## 11. Transferable techniques
1. **Causal-invariant filtering** (Stage 2) — testing that action leads state is a cheap, dataset-agnostic corruption detector, and it found an 81% failure rate in a widely used public subset.
2. **Correct rather than discard** (Stage 4) — convention mismatches are a coordinate-frame bug, not noise.
3. **Render-and-compare video–state check** (Check 2) — URDF projection vs. SAM3 mask is a strong cross-modal integrity test.
4. **Speed-distribution matching** when mixing human and robot streams.
5. **Keyframe protection** during static-segment pruning.
