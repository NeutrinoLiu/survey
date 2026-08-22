# HTD — Learning Versatile Humanoid Manipulation with Touch Dreaming

**arXiv:2604.13015** (v3, Aug 2026) · Carnegie Mellon + UT Arlington + **Bosch Center for AI** (Y. Niu, Fang, B. Chen, S. Zhou, Senthilkumaran, H. Zhang, B. Chen, Qiu, Tseng, Francis, D. Zhao) · Apr 2026 · [site](https://humanoid-touch-dream.github.io)

**One line.** Predicts **future tactile latents as an auxiliary objective inside behaviour cloning** — no separate tactile pretraining stage, targets from an EMA encoder — and shows latent prediction beats raw tactile prediction by 30% relative.

## 1. The humanoid-specific problem

*"Unlike fixed-base manipulation, humanoid manipulation is physically coupled with torso posture, base motion, and foot-ground stability, so **uncertain hand-object contact can affect not only local manipulation accuracy but also whole-body execution**."*

Their survey finds *"few systems combine whole-body control, full end-effector dexterity, and touch sensing/modeling in a single platform."*

## 2. System

**RL-based lower-body controller** as a stability backbone (with a detailed published reward set — feet air-time, stumble penalty, standing contact, flat orientation, feet distance, **centre-of-gravity alignment with the ankle midpoint**, **knee stance width near 0.27 m**), plus **VR teleoperation**, upper-body IK, dexterous hand retargeting, and distributed tactile sensing.

The design intent is operator-facing: a stable execution backbone *"while allowing the operator to focus on task intent and dexterous interaction."*

## 3. Model — Humanoid Transformer with Touch Dreaming

A multimodal encoder–decoder Transformer treating **touch as a core modality** alongside multi-view vision and proprioception, trained in **a single stage** by behaviour cloning augmented with **touch dreaming**: alongside action chunks, the policy predicts

- **future hand-joint forces**
- **future tactile latents**, with targets from an **exponential moving average target encoder**

The EMA-target design (BYOL/JEPA-style) is what removes the need for *"a separate tactile pretraining stage"* — the representation and the policy co-train.

## 4. Tasks

Five real-world tasks, each isolating a distinct difficulty:
- **A** towel folding — long-horizon, multi-stage deformable manipulation
- **B** book organisation — mixed prehensile/non-prehensile, thin objects with limited grasp affordance
- **C** Insert-T — **3.5 mm clearance**, precision plus reactive adaptation
- **D** cat litter scooping — tool-mediated contact under low-profile constraints
- **E** tea serving — bimanual fetch and **loco-manipulation**, whole-body transport keeping objects balanced

## 5. Results

**+90.9% relative** average success over the stronger per-task baseline.

**The ablation that matters:** *"latent-space tactile prediction is more effective than raw tactile prediction, yielding a **30% relative gain** in success rate."*

That is the fourth independent measurement in this survey of the same effect — [[tactile-wam]] (pixel–contact misalignment), [[ratg]] (raw target costs 25 points on plug insertion), [[tacgen]] (13-point reconstruction-vs-utility gap), and now HTD. **Predicting raw tactile is reliably worse than predicting tactile latents.**

The paper also visualises the dreamed tactile latent of the right middle finger as a normalised heatmap changing with finger-object contact — a rare interpretable view of what a tactile prediction head has learned.

## 6. What it adds that the others don't

**Tactile prediction folded into single-stage behaviour cloning**, with EMA targets replacing a pretraining phase — the cheapest predictive-tactile recipe in this survey. And it is the only entry addressing **loco-manipulation**, where contact uncertainty propagates into whole-body stability rather than only into task success. Compare [[tacforesight]] and [[n0-twam]], which get predictive touch through dedicated world models at considerably higher cost.
