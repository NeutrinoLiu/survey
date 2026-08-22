# Skild Brain — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Skild Brain — "omni-bodied" robot foundation model |
| **Org** | Skild AI |
| **Date** | 2025-09 (omni-bodied post) → 2026 (Skild Brain, $1.4B raise at $14B+) |
| **Artifacts** | `page.html`, `omnibodied.html` (skild.ai blog) |
| **Disclosure level** | ⚠️ **C — company blog.** The simulation-side data design is described with a concrete number; the real-world data engine is named but not specified. |
| **Corpus** | *"a universe with **100,000 different robots**"* trained over *"a **millennium** of simulated time"*; plus sim + human video + teleop + real deployment as a combined engine |
| **Stance** | **Diversity as an anti-memorization mechanism.** Curate the *embodiment* distribution, not the observation distribution. |

## 1. Thesis — design a test the model cannot game

Skild's framing is the most explicitly *anti-overfitting* in the survey, and it locates the problem in the data distribution rather than the model:

> *"most controllers are trained for a specific robot. The AI controlling it **memorizes, or 'overfits' to, the locomotion strategy for that robot**. This is a bit like memorizing the answer to a test — great for passing, but unhelpful for learning how to arrive at the answer. When the AI faces a situation it has never seen before, like jammed motors, broken limbs, or a completely new body, the memorized solution is useless."*

> *"How do we fix this? **We must design a 'test' that the AI cannot game.** One way to do this is to train the AI to control not just one robot, but a **whole multiverse of robots with different bodies**. It cannot memorize the solution for one body, it must find a strategy that works across all of them."*

This is a **data-design argument stated as a curriculum principle**: the corpus is constructed so that the shortcut solution does not exist. Compare Qwen-RobotManip (align representations so data can be pooled) and ABot-M0 (balance sampling so no embodiment dominates) — Skild goes further and *manufactures* the embodiment distribution.

## 2. The embodiment corpus

> *"We created a universe with **100,000 different robots** and trained our AI to control them all. After a **millennia of simulated time**, what emerged was a remarkably resilient, omni-bodied brain."*

The number to notice is **100,000 embodiments** — orders of magnitude beyond the 15 morphologies of Qwen-RobotManip's H2R pipeline or the 20+ of ABot-M0's UniACT. Procedural generation of bodies is the only way to reach that count, which makes the embodiment axis effectively unbounded in a way the real-data axis is not.

**Held-out protocol, stated explicitly:**
> *"for all these experiments, **we excluded all these robots from our dataset** – the model is never trained on any of these robots and is **tested zero-shot**."*

A clean embodiment-level train/test split — the analogue of DYNA-2's held-out robot suite and Hydra-0's excluded IWS tasks, applied to bodies rather than tasks.

## 3. Reported zero-shot behaviours

Each is an *out-of-distribution embodiment* test, and the results double as evidence about what the training distribution must have contained:

| Perturbation | Behaviour |
|---|---|
| **Wrong body inference** | Woken on a quadruped, the brain *"decides to treat this robot as a small humanoid"*; because the body has *"only a passive knob for a leg with a single point of contact"*, it falls — then, given the failed trial **prepended as a prompt**, succeeds on the third try. Skild identifies this as **in-context learning**, *"also observed in large language models"* |
| **Loss of limbs** | Calf cut to thigh — removes 4 DoF and shortens the limb. After 7–8 s of adaptation it *"discovers that large amplitude swings are required at the thigh joint."* A **specialist controller trained on a single robot "fails catastrophically and flips over"** |
| **Broken legs** | Knees locked in software, converting a quadruped into a three-legged robot never seen in training |

Also claimed: real-time adaptation to *"jammed wheels, increased payloads, or entirely new bodies, without retraining or fine-tuning"*, across quadrupeds, humanoids, tabletop arms, and mobile manipulators.

The in-context-learning result is the most interesting from a data standpoint: **failed trials become useful input at inference time**, which is another instance of the survey-wide pattern that failure data has value (π₀.₇'s mistake labels, AGIBOT's retained error-recovery trajectories).

## 4. The real-world data engine (asserted, not specified)

> Skild *"uses **simulation, human videos, teleoperation, and real deployments** as a combined data engine to overcome the traditional scarcity of robotics training data."*

And the morphology argument for including human data:
> the company *"trains models that can work across different morphologies **including human data, since humans are also a form of robot**, vastly expanding the available training set."*

Treating humans as one more sampled embodiment is a natural consequence of the 100K-robot framing — it makes human video not a special case requiring retargeting machinery, but simply another point in an already-vast morphology distribution.

⚠️ No hours, sources, filtering, or annotation detail is published for any of the four real-world streams.

## 5. What is not disclosed
- No description of how the 100,000 robot bodies are generated or how their distribution is controlled (this is the central curation question).
- No sizes or processing for the human-video, teleoperation, or deployment streams.
- No scaling curves; results are qualitative demonstrations, not measured success rates.
- No account of the sim-to-real gap for the locomotion policies.

## 6. Transferable takeaways
1. **Curate the embodiment distribution, not just the observation distribution.** If the corpus contains 100K bodies, per-body memorization is not an available solution.
2. **Design the corpus so the shortcut does not exist** — a stronger form of regularization than any filter.
3. **Hold out embodiments, not just episodes.** Zero-shot-on-unseen-bodies is the right generalization test for a cross-embodiment model.
4. **Procedural embodiment generation is unbounded** in a way real-data collection is not — the cheapest axis on which to buy diversity.
5. **Failure trials are inference-time data.** Prepending a failed attempt as context recovers performance without any weight update.
6. **Humans as just another embodiment** dissolves the human-to-robot transfer problem into the cross-embodiment problem, if your morphology distribution is wide enough to contain them.
