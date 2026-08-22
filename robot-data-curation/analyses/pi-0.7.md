# π₀.₇ — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | π₀.₇: a Steerable Generalist Robotic Foundation Model with Emergent Capabilities |
| **Org** | Physical Intelligence |
| **Date** | 2026-04-16 |
| **Artifact** | arXiv 2604.15483 (`paper.pdf`, `paper.html`); pi.website/blog/pi07 |
| **Disclosure level** | **A- — full paper on methodology; corpus composition described qualitatively, no hours/episode counts published** |
| **Model** | 5B total: 4B VLM backbone (Gemma3-init, 400M vision encoder) + MEM-style video history encoder + 860M flow-matching action expert |
| **Stance** | **Don't filter mixed-quality data — *annotate* it and condition on the annotation.** |

## 1. The central idea: conditioning replaces filtering

Every other pipeline in this survey answers "what do we do about bad data?" with a rejection rule. π₀.₇ answers it differently, and states the failure mode it is avoiding:

> *"when the data contains many different modes that differ in terms of both strategy and task performance, a naïve training process would lead to a model that averages together different modes in the dataset and produces suboptimal results."*

Mode-averaging — not corruption — is the problem. And the fix is not to delete the bad modes but to **label which mode each episode belongs to**, so the model learns `p(action | observation, quality)` rather than `p(action | observation)`:

> *"we address this challenge by annotating the data with detailed context annotations that contain not only information about what to do but also how to do it."*

The explicit framing: *"we do not aim to propose a new architecture or model design, so much as a methodology for enabling VLAs to utilize more diverse data sources."* π₀.₇ is, by its own description, a **data-methodology paper**.

## 2. Sources — deliberately including what others exclude

| Stream | Note |
|---|---|
| Demonstration data across many robot platforms | static & mobile, single-arm & bimanual |
| Diverse environments | in-house lab-like, home-like, **and in-the-wild homes** |
| **Autonomous data from policy evaluations** | i.e. the model's own rollouts, including failures |
| **Human interventions within policy rollouts** | correction signal |
| Open-source robot datasets | — |
| **Egocentric human video** | — |
| Web / non-robot auxiliary | object localization, attribute prediction, VQA, text-only |

> *"Instead of just using high-quality demonstration data, π₀.₇ leverages lower quality demonstrations (including failures) and even autonomous data from prior models."*

**Evaluation hygiene:** *"We exclude autonomous data collected in any generalization-focused evaluation task from training"* — an explicit train/eval leakage guard, necessary once your own rollouts feed back into the corpus.

## 3. The annotation schema — "episode metadata" as the cleaning layer

This *is* the pipeline. Each training episode is tagged with:

| Field | Definition | Provenance |
|---|---|---|
| **Overall speed** | Episode length in timesteps, **discretized into 500-step bins** (e.g. 1750–2250 → "2000 steps") | Ground truth (measured) |
| **Overall quality** | Task execution quality, **integer score 1–5**, 5 = highest | **Human annotation** |
| **Mistake** | Boolean per action segment — did the robot err here (failed grasp, wrong subtask)? | **Human annotation, coarse** |
| **Control mode** | `c ∈ {joint, ee}` — text identifier for the action space | Ground truth |

Note the pragmatism: speed is free (measured), and it *correlates* with quality — *"Often faster speed also corresponds to higher quality, e.g., the episode has fewer mistakes."* So one cheap automatic signal partially substitutes for expensive human quality labels. Quality and mistake labels are explicitly **coarse** human annotations, not a per-frame audit.

**Control mode as a prompt token** is a neat alternative to Qwen-RobotManip's FK-consistency correction stage: rather than harmonizing joint-space and EEF-space data into one convention, both are kept and the convention is *declared in the prompt*.

## 4. The other context modalities

- **Expressive language commands** — data collected across diverse tasks/scenarios, then *"annotate the segments with detailed textual descriptions"* at **subtask** granularity, not just task granularity.
- **Subgoal images** — a future frame `g*_t = o_t^end` serving as visual goal. Crucially, the subgoal-training subset `D_g` is *"a subset of segments annotated with especially high-quality subtask labels"* — a **quality-gated sub-corpus** for the one objective that needs precise segmentation. At runtime subgoals are produced by a lightweight world model built on **BAGEL** (14B mixture-of-transformers), itself trained on web data, egocentric human video, and other video — so non-robot data enters through the world model as well as the policy.

### Full prompt format
```
<Multi-view observation><Multi-view subgoals>
Task: peel vegetables. Subtask: pick up the peeler.
Speed: 8000. Quality: 5. Mistake: false. Control Mode: joint.
<Proprioception>
```

**Random dropout of each context component during training** makes every field optional at inference, so the same model works with or without metadata, subgoals, or subtask labels.

**At runtime you prompt for the behavior you want**: *"the model can then be instructed to perform the task at high speed, with high quality, and without mistakes, through metadata prompting."* The bad data trains the model; you simply never ask for it back.

## 5. Scaling evidence — the key result

Fig. 18 is the most directly relevant scaling result in this survey to the question "does cleaning matter?":

> **Left panel:** *"π₀.₇ (with metadata) can continuously improve its performance when it is trained on larger datasets, **even when the average quality of the data actually decreases**. By contrast, without training on rich conditioning information, π₀.₇ (without metadata) actually can **degrade** in performance as more lower quality data is introduced."*

> **Right panel:** *"When π₀.₇ is trained without our robot data with the highest task diversity, its performance degrades substantially"* — task diversity, specifically, drives compositional generalization.

Read carefully, this says: **quality filtering and conditioning are substitutes.** If you can condition, you can keep the low-quality data and still scale. If you cannot, quality becomes a hard constraint on how much data you can absorb. This is the mirror image of Qwen-RobotManip's finding that *alignment* is the prerequisite for scaling — both identify a representational precondition without which added data stops helping.

**Emergent downstream behavior:** π₀.₇ performs unseen short-horizon tasks out of the box (french-press plunger, scooping rice, wiping objects, spinning articulated items) with no task-specific robot data. For unseen *long-horizon* tasks (loading/unloading an air fryer, toasting a bagel — up to 5 minutes, multi-stage), it is **coached in language** step by step, and the coaching transcript can then be finetuned into a high-level policy. Reported zero-shot success 60–80%; in-distribution >90%. In one demonstration, language coaching lifted air-fryer success from ~5% to ~95% in half an hour of prompt tuning — i.e. **new "training data" supplied as prompts rather than demonstrations.**

## 6. What they do not do
- **No rejection cascade at all.** No signal-quality filters, no dedup, no episode exclusion (beyond the eval-leakage guard).
- No corpus statistics published — hours, episode counts, and per-source composition are not disclosed.
- Quality/mistake labels are human and coarse; no inter-annotator agreement or automation reported.
- Relies on having enough *variance* in quality for the metadata to be learnable — *"The data diversity (e.g., episodes of varying speed) provides the necessary signals for the model to learn to correlate such metadata with the target action."* A uniformly clean corpus would make the metadata useless.

## 7. Transferable takeaways
1. **Condition, don't filter.** Quality as an input feature turns a rejection decision into a runtime choice, and lets a corpus keep growing past the point where filtering would have capped it.
2. **Use free proxies for expensive labels** — episode duration is measured for nothing and correlates with quality.
3. **Declare conventions in the prompt** (`Control Mode: joint|ee`) instead of harmonizing them in the data.
4. **Randomized context dropout** makes every annotation optional, so partial metadata coverage is not a blocker — the same problem Hydra-0 solves with sampling-probability renormalization.
5. **Gate only the objective that needs it** — subgoal training uses the high-quality-label subset while the main policy trains on everything.
6. **Guard against feedback-loop leakage** once autonomous rollouts re-enter training.
