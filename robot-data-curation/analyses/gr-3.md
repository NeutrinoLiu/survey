# GR-3 / GR-RL (ByteDance Seed) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **GR-3 Technical Report** (arXiv 2507.15493, 2025-07) → **GR-RL** (2026, latest in stream; `bytedance-gr-rl/page.html`) |
| **Org** | ByteDance Seed |
| **Disclosure level** | **A for GR-3** (full technical report). **B for GR-RL** (Seed blog post). |
| **Corpus** | Robot trajectories + web-scale vision-language + human trajectory data (VR-collected) |
| **Stance** | **Control the data distribution at collection time with a scheduler**, then filter, then co-train. |

## 1. Three-stream training recipe

> *"We train the GR-3 model on a mixture of data sources, including **robot trajectory data** for imitation learning, **web-scale vision-language data** for co-training, and **human trajectory data** for few-shot generalization."*

Each stream is assigned a distinct capability target:
1. Robot trajectories → task execution
2. VL data → *"generalize to novel objects, environments, and instructions"*
3. Human trajectories → few-shot adaptation

## 2. The distinguishing mechanism — a collection scheduler

Most works balance a corpus *after* collection, by sampling. GR-3 balances it *during* collection:

> *"[during] trajectory collection, the system **generates a new configuration for the teleoperator to arrange the environment accordingly**. The implementation of the scheduler enables us to **effectively manage the overall data distribution and thoroughly randomize the collected data**, greatly enhancing the richness and variability of the dataset."*

This closes the loop between the intended distribution and the collected one at zero marginal cost — an operator is going to reset the scene anyway, so the scheduler decides *how*. It is the cheapest possible form of diversity control, and it sidesteps the long-tail rebalancing problem that ABot-M0 has to solve with task-uniform sampling and Gini analysis.

## 3. Post-collection filtering
> *"post-collection quality checks are conducted to refine the dataset by **filtering out invalid and low-quality data**."*

⚠️ Criteria and rejection rates are not specified.

## 4. Viewpoint hygiene — an explicit anti-shortcut measure
> *"Previous work indicates that policies can take advantage of **spurious correlations from multiple viewpoints**"*

GR-3 treats multi-view camera configurations as a **shortcut-learning risk in the data**, and designs against it. This is a rarely-articulated failure mode: with several fixed cameras, a policy can infer scene identity (and therefore the right action) from view geometry rather than from task-relevant content. It is a data-design issue, not a model issue.

## 5. VL co-training corpus — filtering *and* re-annotation
> *"We curate a large vision-language dataset from a mixture of data sources… covering **image captioning, visual question answering, image grounding, and interleaved grounded image captioning**. We also develop a **filtering and re-annotation pipeline** to improve the quality of the dataset for effective co-training."*

Both operations, as in LingBot-VA 2.0 (relabel) and Cosmos 3 (filter): drop what is unsalvageable, rewrite what is merely bad.

The stated payoff is bidirectional: co-training *"not only helps GR-3 maintain the strong vision-language capabilities from the pre-trained VLM, but also **enables the action DiT to leverage these capabilities in action prediction**."* This is the same anti-forgetting rationale as Qwen-RobotManip's 28M-sample VL stream and π₀.₇'s web-data auxiliary sources.

## 6. Human trajectory collection — quality via control architecture

Two mechanisms that improve data quality by improving the *collection instrument*:

- **Whole-body compliance control**: *"treats all degrees of freedom as a holistic structure to retarget arbitrary teleoperated human motion to feasible robot motion. **Manipulability optimization, singularity avoidance, and physical joint limits are simultaneously addressed within a real-time optimal control problem**… This generates fluid and continuous [trajectories] for policy training."*
- **Whole-body teleoperation** via Meta Quest VR, mapping human motion directly to robot end-effectors.

The compliance controller is effectively a **validity filter applied at record time**: singularities and joint-limit violations never enter the dataset because the controller cannot produce them. Compare EgoScale's constrained retargeting optimization — same principle, applied to teleoperation rather than to video reconstruction. *"The compliant force controller enables highly dynamic motion and physical interaction with the environment, enhancing safety and **data collection efficiency**."*

## 7. Generalization evaluation — including an "invalid task" set

For the generalization study: **35K robot trajectories, 101 objects, 69 hours**, annotated with `"put A into B"` templates.

Six test conditions, of which two are unusually well-designed data probes:
- **Novel Destinations** — *"put the fork into the rubbish bin"*, a pairing absent from training. Baseline π₀ *"puts objects into containers that appear together with the objects in the training data instead of following the given instructions"* — a direct demonstration of **co-occurrence shortcut learning** caused by the training distribution.
- **Invalid Tasks** — instructions that cannot be satisfied by the observation (e.g. "put the blue bowl into the plastic box" when no blue bowl exists). *"The trial is considered successful only if the model **refrains from manipulating any**"* object.

The Invalid Tasks setting is a rare and valuable evaluation: refusal is a behaviour that a corpus of only-successful, only-valid demonstrations cannot teach. GR-3 *"strictly follows instructions in all six test sets"* and *"is able to refrain from performing wrong tasks."*

## 8. GR-RL (latest in stream)
Real-world reinforcement learning applied to high-precision manipulation. Reported: **shoe-lacing success 45.7% (GR-3, supervised) → 83.3% (GR-RL)** — nearly 70% fewer failures. The data-side significance is that the improvement comes from *on-policy experience*, moving this stream toward the same autonomous-data regime as π*₀.₆/RECAP and 1X's flywheel.

## 9. What they do not do
- Filtering criteria and rejection rates unpublished.
- No corpus hour totals or mixture proportions for the main pretraining run.
- No dedup.
- No fitted scaling curve.

## 10. Transferable takeaways
1. **Schedule the collection.** Generating environment configurations for the teleoperator controls the data distribution at source, for free.
2. **Treat multi-view setups as a shortcut risk**, and design the camera configuration as a data decision.
3. **Filter *and* re-annotate** the VL co-training corpus — the two operations address different defects.
4. **Put the validity constraints in the controller.** Singularity avoidance and joint-limit handling at record time mean invalid trajectories never need filtering.
5. **Evaluate refusal.** An "invalid tasks" test set measures something a success-only corpus cannot teach, and exposes co-occurrence shortcuts that in-distribution benchmarks hide.
