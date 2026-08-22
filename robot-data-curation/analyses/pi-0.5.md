# π₀.₅ — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | π₀.₅: a VLA with Open-World Generalization |
| **Org** | Physical Intelligence |
| **Date** | 2025-04 (arXiv 2504.16054) |
| **Artifact** | `paper.pdf`, `paper.html` |
| **Disclosure level** | **A — full paper; corpus composition qualitative** |
| **Stream** | π₀ → **π₀.₅** → [π*₀.₆/RECAP](../pi-star-0.6-recap/dataprocess.md) → [π₀.₇](../pi-0.7/dataprocess.md) (latest) |
| **Stance** | **Co-training on heterogeneous tasks and abstraction levels is what produces open-world generalization.** |

## 1. The thesis — breadth of *abstraction*, not just breadth of data

> *"the diversity of situations that a robot might encounter in the real world requires **more than just scale**: we need to design training recipes that can provide the **breadth of knowledge that will allow robots to generalize at many levels of abstraction**."*

This is the π-series' recurring data argument, and π₀.₅ is where it is first made explicit. Scale alone is insufficient; the corpus must span **abstraction levels** — from web semantics down to motor commands — because generalizing to an unseen kitchen requires competence at every level simultaneously.

## 2. The mixture

> *"π₀.₅ uses data from **multiple robots, high-level semantic prediction, web data, and other sources**… Our system uses a combination of **co-training and hybrid multi-modal examples that combine image observations, language commands, object detections, semantic** [subtask prediction, and low-level actions]."*

The distinctive construct is the **hybrid multi-modal example**: a single training sample containing observations, commands, detections, semantic subtask labels, and actions together, rather than separate samples for separate objectives. This forces the model to learn the *links* between abstraction levels rather than each level independently — and it is the direct ancestor of π₀.₇'s composite prompt (`Task / Subtask / Speed / Quality / Mistake / Control Mode`).

## 3. High-level semantic prediction as a data type
π₀.₅ trains the model to **predict subtasks** ("pick up the plate") and then emit low-level actions for them. Concretely, in the kitchen demonstration: *"The model is given general tasks (close the cabinets, put the items in the drawer, wipe the spill, and put the dishes in the sink), which it performs by both **predicting subtasks to accomplish** and **emitting low-level actions**."*

The data requirement is **subtask-level annotation** of long trajectories — the same annotation layer that AGIBOT WORLD 2026, Galaxea, ABot-M0, and LingBot-VA 2.0 each independently build. It is the single most commonly required non-trivial annotation in this survey.

## 4. Generalization claim
> *"bedrooms in new homes that were **not present in the training data**, performing complex multi-stage behaviors with durations of **10 to 15 minutes**."*

Whole-home generalization from a corpus that never contained those homes is the result that established the in-the-wild-home data strategy later pursued by Figure (Brookfield), DreamZero (22 environments), and DYNA-2.

## 5. Position in the π lineage — a data-strategy trajectory

| Model | Data-side contribution |
|---|---|
| **π₀** | Flow-matching VLA on diverse robot demonstrations |
| **π₀.₅** | **Co-training across abstraction levels**; hybrid multi-modal examples; web + multi-robot + semantic prediction |
| **π*₀.₆ / RECAP** | **Learned value function grades every trajectory**; integrates demonstrations + interventions + autonomous rollouts |
| **π₀.₇** | **Human quality/mistake/speed metadata as conditioning**; scales on data whose average quality is *falling* |

Read in sequence, this is a coherent research program about **how to keep absorbing data as its quality degrades**: first broaden what counts as data (π₀.₅), then grade it automatically (RECAP), then let the user select the grade at inference (π₀.₇).

## 6. What they do not do
- No corpus statistics — hours, episodes, per-source composition all undisclosed (a constant across the π series).
- No cleaning cascade or filtering rules.
- No dedup, no rejection rates.
- No fitted scaling law.

## 7. Transferable takeaways
1. **Diversity of abstraction is a distinct axis from diversity of scenes or tasks.** A corpus can be large and varied yet still flat in abstraction.
2. **Hybrid multi-modal examples** — one sample carrying observation, command, detection, subtask, and action — teach the links between levels that separate corpora cannot.
3. **Subtask annotation is the highest-leverage non-trivial label** in robot data; nearly every long-horizon system in this survey requires it.
4. **Co-training with web data at pretraining time**, not as an afterthought, is what preserves the semantic generalization the VLM backbone brought.
