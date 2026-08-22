# LingBot-VA 2.0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Native Video-Action Pretraining for Generalizable Robot Control (LingBot-VA 2.0) |
| **Org** | Robbyant / Ant Lingbo Technology (Ant Group) |
| **Date** | 2026-07 |
| **Artifact** | arXiv 2607.08639 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper, §3 "Data Recipe"** |
| **Corpus** | Web-scale image/video (LingBot-Video corpora) + **thousands of hours** robot + **thousands of hours / 65.4K episodes** internal egocentric human + **>50K synthesized human-robot ICL pairs** |
| **Stance** | **Pretrain the whole stack for embodiment.** Causal video-action pretraining *from scratch*, not a fine-tuned video generator. |

## 1. Thesis

> *"The research team pretrains the whole stack for embodiment instead of fine-tuning a video generator. Given the strictly causal nature of temporal dynamics, they adopt a **causal pretraining paradigm, training from scratch to circumvent the catastrophic forgetting** that frequently occurs when adapting bidirectional architectures."*

And the data consequence:
> *"Because its action signal no longer depends on scarce robot demonstrations, LingBot-VA 2.0 acquires **control knowledge at the scale of web video rather than at the scale of robot datasets**."*

Payoff: adapts from **only 10–15 demonstrations**, transfers across embodiments without re-collection, and in some settings executes new tasks zero-shot.

## 2. Three-stage data curriculum

| Stage | Data | Purpose |
|---|---|---|
| 1 | General **text-to-image** pretraining | Appearance priors |
| 2 | General **text-to-video** pretraining | Dynamics priors |
| 3 | **Video-action adaptation** on embodied data | Control |

Stages 1–2 reuse the web-scale corpora curated for **LingBot-Video** — the data pipeline is inherited from a sibling video-generation project rather than rebuilt, a sensible reuse pattern for organizations with both.

## 3. Robot data — relabeling as repair

Retained sources: **AgiBot, RoboMIND, InternData-A1, Open X-Embodiment (incl. DROID), UMI-style datasets, RoboCOIN**, plus *"thousands of hours of internal robot demonstrations."*

> *"All robot data are **reprocessed with a unified annotation pipeline**. Each long trajectory is **segmented into atomic action clips**, and each clip is assigned a **language prompt and a global task instruction using Qwen3.5-397B**. This relabeling step **repairs missing or overly generic prompts in earlier versions** and makes the language supervision more consistent across embodiments and datasets."*

**Wholesale LLM relabeling** rather than filtering out bad labels. Compare Qwen-RobotManip, which *verifies* existing labels with a VLM cascade and drops the inconsistent ones; LingBot **regenerates them all**, which normalizes phrasing style across corpora as a side effect. Two defensible answers to "the language annotations are bad."

## 4. Egocentric human corpus — deliberately balanced by construction

An internal corpus of **thousands of hours / 65.4K episodes**, explicitly balanced:
- **Five tabletop-oriented environment categories**: kitchens, dining tables, vanity tables, office desks, tool benches
- **600+ operators**
- **3,000+ scene-task combinations**
- Skills: object placement, cleaning, refilling, assembly, packaging, stationery use, tool use, cosmetic-item organization

Per-frame annotation: **6-DoF world-frame root poses of both hands + 22 finger-joint angles per hand**, single egocentric RGB stream.

Balancing the corpus *at collection time* across five named environment categories is the same lesson as EgoVerse's structured assignment matrix: diversity you designed for is measurable; diversity you hope for is not.

## 5. Human→robot action harmonization — a better gripper proxy

All actions live in *"a unified cross-embodiment action layout in which channels missing for an embodiment are **zero-padded and masked**"*; the bimanual EEF slice holds, per hand, a **6-DoF root pose + a scalar gripper opening**.

Human hand labels are retargeted into exactly this slice by a map Φ:
- **6-DoF hand-root poses are kept as is, with no conversion**
- Finger configuration `q` → scalar gripper opening `φ(q)`

The key detail, and an improvement on the common thumb–index heuristic:
> *"From the captured finger joints we recover 3D fingertip positions by forward kinematics and take the opening as the distance between the two sides along the closing direction, yielding a **metric aperture that is quantile-normalized per dataset** in the same way as the robot gripper channels. **Measuring this over the whole finger envelope rather than a single thumb-to-index pair keeps it stable when a finger is mistracked.**"*

This is explicitly a **robustness-to-tracking-failure** design choice — contrast DYNA-2's thumb–index aperture and Qwen-RobotManip's thumb/virtual-finger midpoint, both of which are single-pair and therefore fragile to one mistracked digit.

> *"This functional opening is what **replaces the no-op gripper placeholder** used in prior co-training."*

And the honest residual: *"what remains is the **motion gap**—the same tuple carries different distributions and physical meaning for a human hand and a robot end-effector—which the **per-domain action heads absorb**." Separate human and robot action heads over a shared video-action backbone; human heads are **initialized from the robot heads** for stability.

Converted human clips then *"incorporated into the **same unified annotation and filtering pipeline** as the robot corpus"* — one pipeline, two sources.

## 6. The reverse H2R — synthesizing *human* video from *robot* video

The most distinctive data-generation idea here. Where Qwen-RobotManip renders robots into human video, LingBot renders **humans into robot video**, to build in-context-learning pairs:

```
Large-scale robot dataset
  → sample videos guided by a TASK TAXONOMY
  → VLM (Gemini-3.1-Pro) analyses task, generates an IMAGE-EDITING PROMPT
       converting the robot's first frame into a human-manipulation initial observation
       ("First-person egocentric perspective. Two human arms naturally extend from …")
  → first-frame edit (Nano Banana Pro)
  → VLM generates a VIDEO-GENERATION PROMPT
  → video model (WAN-2.6 / Kling-V3) synthesizes the human manipulation video
  → VLM SCORES the generated video on:
        (a) task semantic preservation
        (b) physical plausibility
  → qualified videos PAIRED with their original robot trajectories → ICL sample
```

Deliberately unconstrained: *"This editing process imposes **no constraints on viewpoint, scene, or object instances**, requiring only the preservation of the original robot task semantics."* Maximizing appearance variance while pinning semantics.

**Result: >10 dataset sources, >5K robotic tasks, >50K human-robot pairs.**

Note the **two-criterion VLM gate** — semantic preservation *and* physical plausibility. Generative augmentation is filtered by a judge, closing the loop that RoboCurate closes with simulator replay. Both works independently conclude that **synthetic data must be verified, not merely generated.**

## 7. What they do not do
- No published rejection rates for the "filtering pipeline" applied to robot and human corpora.
- Web-scale image/video curation is inherited from LingBot-Video and not described here.
- The ICL gate is a single VLM with no cross-model adjudication.
- No fitted data-scaling curve.

## 8. Transferable takeaways
1. **Relabel rather than filter** when language annotations are the defect — an LLM pass normalizes style across corpora as a bonus.
2. **Measure gripper aperture over the whole finger envelope**, not a single digit pair, so one mistracked finger doesn't corrupt the label.
3. **Balance the human corpus at collection time** across named environment categories, operators, and scene-task combinations.
4. **Robot→human synthesis** is a viable inverse of the usual H2R direction, and is the natural way to build in-context human-demonstration pairs at scale.
5. **Gate generative augmentation on two axes** — semantic preservation and physical plausibility — because those failures are independent.
6. **Absorb the irreducible gap in the architecture** (per-domain action heads) once representation alignment has done all it can.
