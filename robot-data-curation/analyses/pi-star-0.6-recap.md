# π*₀.₆ / RECAP — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | π*₀.₆: a VLA that Learns from Experience (RECAP — RL with Experience & Corrections via Advantage-conditioned Policies) |
| **Org** | Physical Intelligence |
| **Date** | 2025-11 |
| **Artifact** | arXiv 2511.14759 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper** |
| **Corpus** | Pre-training: *"**tens of thousands of hours** of demonstrations from numerous tasks and a variety of different robots"* |
| **Stance** | **Grade every trajectory by learned value, then condition on it.** The direct ancestor of π₀.₇'s metadata conditioning. |

## 1. Why this belongs in a data-curation survey

RECAP is not a filtering pipeline — it is the mechanism that makes *filtering unnecessary*. It answers the question every work ingesting autonomous rollouts must answer: **how do you train on data that contains your own failures?**

> *"we train a **value function that evaluates progress toward successful task completion**, and then use this value function to **estimate the advantage of each action in the dataset**. By **conditioning the policy on an improvement indicator based on this advantage**, we can obtain an improved policy."*

Where π₀.₇ later uses **human-annotated** quality scores (1–5) and mistake flags, RECAP derives the equivalent signal **automatically** from a learned value function. The lineage is direct: RECAP (learned advantage as conditioning) → π₀.₇ (human metadata as conditioning). Both refuse to discard bad data; they label it and let the model condition on the label.

## 2. Three data sources, integrated rather than filtered

> *"our method uses **both expert interventions and fully autonomous experience**, resulting in an RL-based framework that **integrates multiple data sources**."*

| Source | Nature | Conventional treatment | RECAP treatment |
|---|---|---|---|
| **Demonstrations** | Human, high quality | Keep | Advantage-graded |
| **Expert interventions** | Human corrections *inside* a policy rollout | Usually discarded or handled specially | Advantage-graded |
| **Autonomous rollouts** | The policy's own experience, including failures | Usually filtered out | Advantage-graded |

Expert interventions are a particularly valuable and under-collected data type: they are precisely the state–action pairs where the policy was about to fail and a human corrected it — maximal information density per frame. AGIBOT's retained error-recovery trajectories are the demonstration-side analogue.

## 3. The value function as an automatic quality scorer

The mechanism, in curation terms:
- Returns `R_t(τ)` are **discretized into B = 201 bins** (`R_t^B`), and the value function is trained by **minimizing cross-entropy** over trajectories in the current dataset `D`.
- *"This is a **Monte Carlo estimator for the value function of the policy represented by the dataset D** (i.e., the behavior policy π_ref)."*
- A continuous value is recovered as an expectation over the bins: `V^{π_ref}(o_t, ℓ) = Σ_{b∈[0,B]} p_φ(V = b | o_t) · v(b)`, and the advantage follows.

**Why this matters for data practice:** the value function is a *learned, per-timestep, task-conditioned quality score* obtained without any human annotation. It is:
- **Automatic** — no annotators (contrast π₀.₇'s coarse human labels)
- **Per-timestep**, not per-episode — a mostly-good episode with a bad segment is scored correctly at both parts
- **Task-conditioned** — quality is relative to the instruction, not absolute
- **Self-referential** — it scores relative to the behavior policy in the dataset, so it improves as the dataset improves

This is arguably the most principled "data quality score" in the entire survey, because it is defined in terms of the thing that actually matters (progress toward task completion) rather than in terms of a proxy (smoothness, aesthetics, caption agreement).

## 4. Training structure
Pre-training performs value-function fitting and advantage-conditioned policy training over *"our entire pre-training dataset… tens of thousands of hours of demonstrations from numerous tasks and a variety of different robots."* The full RECAP loop (including on-robot data collection) is then run in the fine-tuning phase.

Conditioning uses an improvement indicator in the manner of classifier-free-guidance-style approaches, *"extend[ed] to pre-train and fine-tune a large-scale generalist VLA policy, incorporating a variety of data sources (including demonstrations, interventions, and autonomous policy roll-outs)."*

## 5. Position in the survey

RECAP closes a loop that several other entries open but do not resolve:
- **1X** describes an on-policy data flywheel from 20,000 deployed NEOs but publishes nothing about filtering it. RECAP is what that filter would look like.
- **π₀.₇** demonstrates that quality-conditioned training lets performance keep improving *as average data quality falls*. RECAP supplies the automatic version of that quality signal.
- **GEN-0** notes that some corpora are better for SFT and others for RL post-training. RECAP's advantage conditioning is the mechanism that makes mixed-quality corpora usable for both.

## 6. What they do not do
- No corpus composition table, no per-source hours.
- No conventional cleaning pipeline (by design — quality is handled in the objective).
- Value-function quality is itself data-dependent; failure modes of the scorer on out-of-distribution data are not characterized.
- Requires task success to be well-defined enough to fit a return, which limits applicability to open-ended human video.

## 7. Transferable takeaways
1. **A learned value function is a free, per-timestep, task-conditioned data-quality score.** No annotators, no heuristics, and it measures the right thing.
2. **Condition on advantage rather than filtering by it** — the policy learns what good and bad look like, and you ask for good at inference.
3. **Collect expert interventions deliberately.** Corrections inside a rollout are the highest-information-density supervision available, and are usually thrown away.
4. **Discretize the return into bins and fit with cross-entropy** — a stable, well-behaved way to estimate value distributions over trajectory datasets.
5. **Autonomous rollouts are a data source, not a contaminant**, once you have a grading mechanism. Every deployment flywheel needs one.
