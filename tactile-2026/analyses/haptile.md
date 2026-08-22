# HapTile — A Haptic-Informed Vision-Tactile-Language-Action Dataset for Contact-Rich Imitation Learning

**arXiv:2606.04825** (v2) · King's College London + Huawei Noah's Ark Lab + UCL (Alian, Y. Zhao, Gu, X. Zhang, Z. Chen, Mower, Bou-Ammar, S. Luo) · Jun 2026 · [site](https://haptile-dataset.github.io/)

**One line.** Puts tactile sensing at **both ends of the teleoperation loop** — fingertip sensors on the robot, and **haptic feedback to the human operator** — on the argument that demonstration quality itself depends on the demonstrator being able to feel.

## 1. The two-level design

Most tactile datasets instrument the robot. HapTile also instruments the *operator*: *"existing teleoperation pipelines rarely provide haptic feedback to the operator, despite its established role in demonstration quality and manipulation stability."*

- **Robot side** — custom-designed fingertip vision-based tactile sensors
- **Operator side** — haptic feedback integrated directly into the teleoperation controller, *"enabling the operator to perceive contact interactions in real time"*

A second design distinction, stated against the field: *"language instructions are used as policy conditioning signals rather than descriptive metadata."* Their comparison table marks several prior datasets (DROID, RH20T, TVL, MMWand, TaF-VLA) with **✓\*** — language present, but only as description. That distinction matters for whether a dataset can train a VLA at all.

## 2. Dataset

| | |
|---|---|
| Demonstrations | **1,726** |
| Tasks | **38** |
| Skills | **9** — pick-and-place, pressing, wiping, turning, folding, stacking, compliant handling |
| Objects | diverse mechanical/geometric properties, including **YCB** items |
| Platform | **UR5e** — deliberately standard, for reproducibility |

Tasks *"span... varying levels of tactile dependency"* — a design choice that, like [[deco]]'s inclusion of tasks where tactile does not help, makes the dataset able to support negative results.

## 3. Results

Benchmarked with **Diffusion Policy** and **π₀**.

The performance comparison is instructive about dataset scale: *"Diffusion Policy, being more compact, learns effectively from 50 demonstrations, while the larger pretrained π₀ struggles to align novel tactile inputs under limited fine-tuning data. Notably, π₀ performs more consistently with the structured V+TM representation."*

That is a concrete data-regime finding — **at 50 demonstrations, a small from-scratch policy beats a large pretrained one at absorbing a new modality**, unless the tactile input is given structured form. It is the small-data counterpart to [[at-vla]]'s result that high-dimensional tactile perturbs pretrained representations most.

## 4. Stated limitations, all honest

- **Single laboratory scene** — objects and clutter are diverse but scene layout, lighting and background are not. In-the-wild collection is needed.
- **Physical collection does not scale** — simulation with realistic physics is proposed, with the sim-to-real gap acknowledged.
- **Haptic feedback is binary.** *"The current haptic mechanism operates on force thresholds, distinguishing only two discrete force modes... it does not capture the continuous force variation characteristic of real manipulation. Graduated force levels would more faithfully reflect contact dynamics, particularly for tasks involving fragile, deformable, or surface-sensitive objects."*

That last limitation is worth dwelling on, because it bounds the paper's own premise: if haptic feedback improves demonstration quality, a two-level force signal is a coarse instrument for delivering it.

## 5. What it adds that the others don't

**Closing the haptic loop on the human side.** Every other dataset in this survey treats the demonstrator as a source of actions; HapTile treats demonstration quality as something the *interface* determines, and instruments accordingly. Combined with language-as-conditioning rather than language-as-metadata and a deliberately standard UR5e platform, it is among the more reproducible VTLA datasets — and its comparison table is a useful map of what the alternatives actually contain.
