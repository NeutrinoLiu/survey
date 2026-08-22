# DreamZero — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | World Action Models are Zero-shot Policies (DreamZero) |
| **Org** | NVIDIA GEAR Lab |
| **Date** | 2026-02-19 |
| **Artifact** | arXiv 2602.15922 (`paper.pdf`, `paper.html`); dreamzero0.github.io |
| **Disclosure level** | **A — full paper** |
| **Corpus** | **~500 hours** teleoperation on AgiBot G1 · **7,193 episodes** · **22 environments** |
| **Stance** | **Diversity over repetition.** *"prioritize breadth and utility over repetition during data collection."* |

## 1. The collection philosophy — stated as a deliberate departure

> *"Our data collection philosophy differs from that of existing VLAs. While recent works have shown that VLAs can learn effective policies from moderate-sized datasets, these approaches typically **rely on structured, task-focused demonstrations to ensure consistent behavior**."*

The hypothesis that licenses the departure is architectural:

> *"We hypothesize that **learning to only predict actions without encoding the knowledge about future world states makes it challenging to leverage highly heterogeneous, non-repetitive data effectively**, as the model must implicitly infer dynamics from noisy state-action pairs. In contrast, we hypothesize DreamZero's **world modeling objective enables effective learning from diverse demonstrations**, allowing us to prioritize breadth and utility over repetition."*

This is the survey's most explicit statement of a principle that recurs everywhere: **the training objective determines what data you can afford to collect.** A pure action-prediction objective needs consistent demonstrations; a world-modeling objective can absorb messy, non-repetitive ones. Compare Being-H0.7 (latent world-action → only needs temporal coherence) and DYNA-2 (video co-training is the only recipe that keeps scaling).

## 2. Corpus profile — long-horizon by design

Collected on **AgiBot G1**:
| Property | Value |
|---|---|
| Total | **~500 hours**, **7,193 episodes** |
| Environments | **22 unique** — homes, restaurants, supermarkets, coffee shops, offices |
| Mean episode length | **4.4 minutes** |
| Mean subtasks per episode | **42.4** |

> *"significantly longer-horizon than typical robotic manipulation datasets"* (DROID, BridgeData).

The skill distribution is chosen from deployment requirements rather than benchmark convenience: *"navigation enables movement between workspaces, while torso adjustments allow"* whole-body repositioning. Collecting 42-subtask episodes rather than 42 separate single-subtask episodes preserves the *transitions* between subtasks — precisely the data that a segmented corpus destroys and that long-horizon execution needs.

## 3. The central data finding

> *"We further find that **diverse distribution of the training data is essential for generalization, outperforming multi-task repetitive data with the same amount of hours**."*

An **hour-matched comparison** — diversity vs. repetition at equal volume. This is the correct experimental design for a data-composition claim (cf. AXIS's volume-matched RoboCasa365 baseline), and it isolates composition from quantity.

Two supporting findings:
- *"higher-quality video predictions… directly translates to superior downstream action execution—indicating that **policy performance is fundamentally tied to video generation quality**."* The world-modeling objective's quality is a *measurable proxy* for policy quality — a cheap offline metric, like EgoScale's validation loss.
- *"autoregressive architectures lead to **smoother robot motions and higher modality alignment** between predicted videos and executed actions."*

## 4. Zero-shot result and its data implication
> *"Despite only being trained on ~500 hours of real-world data, DreamZero shows non-trivial performance on **Genie Sim 3.0**, a simulation benchmark comprised of 100 different tasks, **without being explicitly trained on the 10k hours of simulation training data**."*

500 real hours transferring to a 100-task benchmark whose 10K-hour training set was never touched is a strong argument that *diversity of environment and horizon* substitutes for volume.

## 5. Architecture note relevant to data handling
> *"For robot training data that contains multiple views, we **concatenate all views into a single frame** instead of making architectural changes to the backbone model."*

Multi-view handled as a *data-layout* decision rather than a model change — minimal added parameters (state encoders, action encoders, decoders) *"to retain the generalization capability of video models."* Worth noting against GR-3's warning that multi-view configurations invite spurious correlations; DreamZero's concatenation makes all views one image, which neither exploits nor mitigates that risk explicitly.

## 6. What they do not do
- No filtering/cleaning cascade described — the corpus is self-collected on one platform, so the ABot-M0/Qwen-RobotManip class of cross-source defects does not arise.
- No dedup, no quality gating, no rejection rates.
- 500 h is small; the diversity-over-repetition finding is established at that scale only.
- No fitted scaling law.

## 7. Transferable takeaways
1. **Choose the objective first, then the collection policy.** World modeling buys the ability to collect messy, non-repetitive data; action-only prediction does not.
2. **Compare diversity against repetition at matched hours.** Any claim that composition matters requires holding volume fixed.
3. **Collect long episodes, not segmented subtasks.** 42 subtasks in one 4.4-minute episode preserves inter-subtask transitions that separate recordings destroy.
4. **Derive the skill distribution from deployment**, not from the benchmark — navigation and torso adjustment appear because real workspaces require them.
5. **Video-prediction quality is a usable offline proxy** for downstream policy performance, which makes data ablations cheap.
