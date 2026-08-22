# ABot-World-0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | ABot-World-0: Infinite Interactive World Rollout on a Single Desktop GPU |
| **Org** | AMAP CV Lab, Alibaba Group |
| **Date** | 2026-07-21 |
| **Artifact** | arXiv 2607.19191 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A+ — the most granular quality-check taxonomy published in this survey: 14 named checks across 6 named dimensions, with an explicit hard-reject vs. soft-flag policy.** |
| **Sources** | **AAA game data · simulation engine data · real-world internet video** |
| **Stance** | **"data production as an integral part of model development rather than as a one-time preprocessing stage."** |

## 1. Thesis — the data distribution is actively shaped by the model's weaknesses

> *"This turns dataset construction into a **closed-loop part of world-model development**: the data distribution is **not merely collected once, but is actively shaped to expose the motion, viewpoint, environment, and action regimes in which the model remains weak**."*

**WorldExplorer** performs *"agent-driven collection guided by training feedback"* and *"reallocates collection effort from training feedback"* — a 24/7 autonomous collection loop that targets the model's current failure modes. This is the same closed loop GEN-0 runs across data-foundry partners, but automated and inside a simulator, so the loop closes in hours rather than quarters.

## 2. Three complementary sources, each with a named weakness

| Source | Strength | Stated weakness |
|---|---|---|
| **AAA game data** | *"exact inputs"* — ground-truth control signals from the runtime API | *"can be stylistically narrow"* |
| **Simulation engine** | Deterministic labels from designed trajectories | — |
| **Real-world internet video** | Visual realism, unbounded diversity | Actions must be estimated; carries noise |

Explicitly acknowledging each source's failure mode, then combining them so the weaknesses do not overlap, is the correct way to design a mixture.

## 3. Quality filtering — 14 checks, 6 dimensions, 3 stages

> *"Raw collected data inevitably contains quality issues ranging from technical artifacts (frame drops, resolution mismatches) to semantic problems (user interface overlays, death sequences, geometry glitches)."*

**Six quality dimensions:** (1) file integrity, (2) visual validity, (3) geometric consistency, (4) game state correctness, (5) action-label alignment, (6) metadata quality — supplemented by VLM-based semantic assessment.

**Critically, the stages are ordered by cost:**
> *"Checks execute sequentially—**early-stage format validation enables rapid rejection before computationally intensive analyses**."*

### Stage 1 — File integrity (fast, deterministic, hard reject)
File pair existence · frame count consistency · resolution compliance · frame drop detection.
> *"Format-level errors trigger **clip-level rejection** as they render samples unusable."*

### Stage 2 — Visual validity, geometric consistency, game-state correctness
| Check family | Detects |
|---|---|
| **VLM-based screening** | UI overlays, loading screens, popups, rendering anomalies |
| **Geometric anomaly detection** | **vertical displacement jumps** (terrain clipping), **camera-through-object analysis** |
| **Game state signal processing** | **death-sequence excision**, cutscene removal, map-boundary removal |
| **Action-label alignment verification** | Label/video desynchronization |

Note the domain-specific ingenuity: *death sequences* and *cutscenes* are segments where the action–video causal relationship breaks down entirely (the player's inputs stop mattering). Excising them is exactly analogous to Qwen-RobotManip's state–action causality filter, discovered independently in a completely different data domain.

> *"Third-person character visibility is **assessed and flagged (not rejected)** for downstream weighting."*

### Stage 3 — Metadata quality (soft signals only)
> *"Surviving clips receive metadata annotations—**action-pose consistency scores, screen color shift flags**—that serve as **soft signals for training-time sample weighting and curriculum scheduling rather than hard rejection**, allowing the model to still learn effectively from **imperfect but informative** training samples."*

## 4. The hard/soft distinction — the transferable idea

ABot-World-0 draws the cleanest line in this survey between two categories of defect:

| Defect class | Policy | Rationale |
|---|---|---|
| **Structural / unusable** (corrupt files, wrong resolution, dropped frames, broken action-video causality) | **Hard reject** | The sample cannot serve the objective at all |
| **Degraded but informative** (partial character occlusion, colour shift, imperfect action-pose consistency) | **Flag → sample weight + curriculum position** | The sample still carries signal; discarding it costs coverage |

This unifies what other works do piecemeal: Cosmos 3's major/minor artifact split, π₀.₇'s quality-conditioning, ACE-Ego-0's reliability weighting, and DYNA-2's two-tier routing are all instances of "don't binary-gate what you can weight." ABot-World-0 states it as policy and implements both branches in one pipeline.

## 5. Annotation — source-agnostic schema, source-specific extraction

> *"The pipeline is designed to be **source-agnostic where possible, while exploiting source-specific signals wherever they are available**."*

**Action labels** into one canonical format:
| Source | Mechanism | Fidelity |
|---|---|---|
| Game | *"source-native control signals captured directly from the game's runtime API with **ground-truth precision** and synchronized with each video frame"* | Exact |
| Simulation | *"derive deterministically from designed trajectories"* | Exact |
| Internet video | *"estimated poses via **displacement projection and thresholding**"* | *"carry estimation noise but enable a substantial expansion of training data diversity"* |

The trade-off is stated rather than hidden — the same fidelity gradient ACE-Ego-0 handles with reliability weighting, here handled with the Stage-3 soft weights.

**Scene descriptions**: a large VLM generates structured natural-language descriptions capturing *"overall scene composition, environmental characteristics, weather and lighting conditions, and notable dynamic events, **while deliberately omitting camera** [motion descriptions]"*. Deliberately excluding camera information from text is a thoughtful anti-leakage measure: camera motion is controlled by the *action* channel, so letting it appear in the text conditioning would let the model shortcut around the action input.

Sampling: **stratified sampling** over the curated pool.

## 6. Result
Turns a single **NVIDIA RTX 5090** into a real-time interactive world simulator: **720p, up to 16 FPS, 1.2 s action-to-first-frame latency, ~19 GiB peak VRAM**, infinite action-conditioned rollout (24 h continuous). Achieved via progressive bidirectional-to-causal distillation (teacher forcing + ODE distillation) plus **LongForcing** to mitigate autoregressive drift.

## 7. What they do not do
- Numeric thresholds for the 14 checks are not published (the taxonomy is, the constants are not).
- No rejection rates per check or per source.
- No dedup stage described.
- Domain is interactive world modeling from games/sim/video, not robot manipulation trajectories — the checks transfer conceptually, not literally.

## 8. Transferable takeaways
1. **Name your quality dimensions and count your checks.** "14 checks across 6 dimensions" is a specification; "rigorous cleaning" is not.
2. **Order checks by cost**, cheap deterministic rejection first — the same principle as ACE-Ego-0 (face detection before HaMeR) and Cosmos 3 (design for low yield).
3. **Hard-reject the structurally broken; flag-and-weight the merely degraded.** Two policies, one pipeline.
4. **Excise segments where the action–observation causal link breaks** (death sequences, cutscenes, loading screens) — the game-domain analogue of state–action desynchronization.
5. **Let training feedback drive collection.** WorldExplorer reallocating effort toward the model's weak regimes is a closed loop most robotics pipelines lack.
6. **Omit from the text conditioning whatever the action channel controls**, or the model learns to ignore the actions.
7. **Keep one canonical action format across sources of wildly different fidelity**, and manage the fidelity gap with weights rather than with exclusion.
