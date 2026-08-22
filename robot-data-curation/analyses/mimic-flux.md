# FLUX-mimic / mimic-video — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **mimic-video** (Video-Action Models) → **FLUX-mimic** (2026-07-29, with Black Forest Labs) |
| **Org** | mimic robotics + Black Forest Labs |
| **Artifacts** | `page.html` (trade press); github.com/mimic-video/mimic-video |
| **Disclosure level** | ⚠️ **C — trade-press coverage and a code repo.** No paper, no corpus description, no pipeline. |
| **Stance** | **Buy physical dynamics from the video corpus, not from robot demonstrations.** |

## 1. The argument — where the physics prior comes from

The clearest industrial statement of the video-vs-VLA data trade:

> *"Most modern robot learning pipelines today are built on Vision-Language-Action models, with the backbone **pre-trained on static image-and-text pairs** – so they must learn **physical dynamics almost entirely from scarce, expensive robot demonstration data**. FLUX-mimic instead builds on a **generative video model that already understands dynamics and behavior from large-scale video pre-training**, then trains an action decoder to predict robot actions directly from that visual prediction."*

Black Forest Labs CEO Robin Rombach states the underlying claim:
> *"To **generate convincing video, a model must learn how the physical world behaves; that same understanding enables acting in it**."*

This is the same bet as Cosmos Policy, LingBot-VA 2.0, DreamZero, and DYNA-2 — and the same one DYNA-2's ablation supports empirically (video co-training is the only recipe whose cross-embodiment performance keeps scaling with action data).

## 2. The data-efficiency claim

> *"depending on task difficulty, the model can be fine-tuned for a specific manipulation task with **as little as 30 minutes of robot data, where prior approaches have required 30 or more hours**."*

A claimed **~60× reduction** in per-task robot data. Placed against the survey's other per-task figures — π₀.₇ and GEN-1 at ~1 h, Gemini Robotics 2 at "a few hours", Ψ₀ at 80 trajectories, DYNA-2's bottle-cap task at ~10 minutes — the field has converged in 2026 on **sub-hour per-task robot data** as the expected cost, with the difference between approaches being *which* pretraining corpus absorbs the burden.

## 3. Why the data-cost shift matters industrially
Deployed with **Audi**, exploring *"how frontier Video-Action Models can reduce deployment time, engineering effort, and robot training requirements for real-world industrial automation"*, cutting deployment cycles *"from months to weeks."*

The relevant point for curation practice: **when per-task robot data drops below an hour, the marginal value of curating task-specific data collapses, and essentially all curation leverage moves to the pretraining corpus.** That is a structural argument for why the 2026 investment is concentrated in web-video and human-video pipelines rather than in teleoperation quality control.

## 4. What is not disclosed
- No pretraining corpus description (inherited from FLUX 3, whose data is likewise undisclosed).
- No filtering, annotation, or quality-control pipeline.
- No benchmark results, baselines, or ablations — the 30-minutes-vs-30-hours claim is unquantified against a named baseline.
- No account of how action-decoder training data is selected.

## 5. Transferable takeaways
1. **A video generative model is a physics prior.** Sourcing dynamics from video pretraining rather than from robot demonstrations is the dominant 2026 strategy, and it relocates the entire curation problem to the video corpus.
2. **When per-task data falls below an hour, curate the pretraining corpus, not the task data.** The economics invert.
3. ⚠️ **Counter-lesson**: partnering on a frontier video backbone means inheriting an undisclosed data pipeline wholesale — including whatever biases and licensing exposure it carries.
