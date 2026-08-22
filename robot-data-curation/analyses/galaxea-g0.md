# Galaxea Open-World Dataset + G0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **Galaxea Open-World Dataset + G0 Dual-System VLA** (arXiv 2509.00576, 2025-09) → **G0.5 / G0Plus** (2026-01, latest in stream) |
| **Org** | Galaxea |
| **Disclosure level** | **A — full paper, dataset and models open-sourced** |
| **Corpus** | **500 hours · 150+ tasks · 50 scenes**, single embodiment, subtask-level language annotations (later reported at ~100K trajectories across 11 environments) |
| **Stance** | **Uniformity as a feature.** One embodiment, collected in real homes and workplaces. |

## 1. Thesis — the critique of pooled cross-embodiment corpora

> *"Existing datasets, exemplified by Open-X Embodiment, are predominantly restricted by their **limited task realism and insufficient environmental richness**. These limitations impair the generalization of trained models when confronted with diverse real-world contexts."*

Galaxea's answer inverts the prevailing strategy. Where OXE, UniACT, and Qwen-RobotManip pool many embodiments and then spend a pipeline harmonizing them, Galaxea collects **one embodiment consistently**:

> *"Uniquely, Galaxea Open-World Dataset was **consistently captured using a single robotic embodiment, thereby ensuring uniformity and reliability**."*

The entire class of defects that dominates other pipelines — inconsistent coordinate frames, mismatched control frequencies, unspecified action semantics, differing FK conventions — simply does not arise. **Single-embodiment collection is the strongest possible form of upstream data cleaning.**

What Galaxea spends its diversity budget on instead: *"**authentic human living and working environments**" — 50 real scenes, 150+ tasks.

## 2. Curation
- *"Comprehensive data filtering and precise language annotations further enrich the dataset"* (⚠️ filtering criteria not detailed).
- **Subtask-level language annotations** throughout — *"paired with precise subtask-level language annotations to facilitate both training and evaluation."* Subtask granularity is what makes the data usable for the dual-system planner/executor split, mirroring π₀.₇'s subtask annotations and AGIBOT's hierarchical labels.

## 3. Three-stage curriculum — and the negative result that matters

| Stage | Data | Purpose |
|---|---|---|
| **1. Cross-embodiment pre-training** | Large-scale unlabeled open-source robot datasets | *"acquire general world knowledge priors"* |
| **2. Single-embodiment pre-training** | **Galaxea Open-World Dataset** | *"specialize in the perceptual-action pairs on the target platform"* |
| **3. Task-specific post-training** | High-quality task demonstrations | Mastery of specific complex skills |

**The key finding, and one of the most important cautions in this survey:**

> *"Notably, **when there is a large embodiment gap between the pre-training platform and the target robot, the benefits of cross-embodiment pre-training diminish or can even degrade the VLA model's performance**, underscoring the importance of the proposed single-embodiment pre-training stage."*

Cross-embodiment pretraining is not free. Past some embodiment distance it is **negative-value**. This is direct empirical support for the mechanisms that Qwen-RobotManip (unified alignment), ACE-Ego-0 (morphology conditioning), and ABot-M0 (embodiment-balanced sampling) build to manage embodiment distance — and a warning to anyone pooling corpora without measuring it.

It also names the function of a mid-stage: **single-embodiment pretraining is the bridge** that lets cross-embodiment priors land on a specific platform, structurally identical to EgoScale's Stage-II aligned mid-training and EgoVerse's "domain-aligned anchor."

## 4. Architecture context
Dual system: **G0-VLM** (System 2, multimodal planning) directs **G0-VLA** (System 1, action execution), running asynchronously at different frequencies. The subtask-level annotations are the supervision that makes the split trainable.

## 5. Downstream use
Galaxea Open-World is consumed as a source by several later corpora in this survey — **Qwen-RobotManip** (~500 h, mobile bimanual), **ABot-M0** (a "high quality" pillar of UniACT), **Qwen-VLA** — which is a practical endorsement of the single-embodiment, high-annotation-quality strategy.

## 6. Latest in stream
**G0.5** — pretrained autoregressive VLA for general-purpose robot control; **G0Plus** released 2026-01-04. ⚠️ A G0.5 technical report exists but the data-side deltas from G0 are not separately documented here.

## 7. What they do not do
- Filtering criteria, rejection rates, and defect taxonomy unpublished.
- Single embodiment caps embodiment diversity by construction — the trade Galaxea explicitly accepts.
- No fitted scaling curve.

## 8. Transferable takeaways
1. **Single-embodiment collection eliminates most cleaning problems upstream.** If you control collection, uniformity is cheaper than harmonization.
2. **Spend the diversity budget where it is scarce** — real living and working environments, not robot models.
3. ⚠️ **Measure embodiment distance before pooling.** Cross-embodiment pretraining can *degrade* performance when the gap to the target platform is large.
4. **Insert a single-embodiment bridge stage** between broad pretraining and task post-training — the same "aligned anchor" pattern that EgoScale, EgoVerse, and Ψ₀ arrive at independently.
5. **Annotate at subtask granularity** if a planner/executor split is intended; task-level labels cannot supervise it.
