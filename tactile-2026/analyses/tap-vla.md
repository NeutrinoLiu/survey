# TAP-VLA — Tactile Annotation Prompting for Vision Language Action Models

**arXiv:2606.29089** · University of Michigan (Van der Merwe, Shehab, Lee, Wi, Dai, Berenson, Fazeli) · Jun 2026 · [site](https://tap-vla.github.io/)

**One line.** Draws the tactile shear field **as arrows on the camera image** the policy already looks at. No new encoder, no new input branch, no architectural change — and it beats every fusion baseline that has the same signal.

## 1. What "tactile" means here

**Shear fields extracted from GelSight**, rendered as **down-sampled, spatially-grounded shear vectors overlaid onto the multi-view RGB images**.

The reasoning is a distribution-shift argument. VLA strength comes from large-scale pre-training, but that data *"contains no tactile signals, and tactile data is itself too heterogeneous and scarce to pre-train at a comparable scale"* — F/T sensors emit 6D wrenches, GelSight emits RGB, and existing datasets are small and sensor-specific. So tactile must be grafted on at fine-tuning, and every prior graft (dedicated encoders, extra input branches) *"induces a distribution shift away from the pre-training the policy relies on."*

The alternative: put tactile in the **VLA's native observation space**. The lineage is visual prompting for VLMs — red circles steering CLIP, Set-of-Mark on GPT-4V, RT-Trajectory's trajectory sketches, TraceVLA's point-trajectory overlays. TAP-VLA is the tactile member of that family.

**A practical detail that matters:** vector magnitudes are scaled by the **95th percentile of shear magnitude computed per task during training**, and clipped at **twice** that percentile *"to avoid spurious shears causing unexpected visuals."* Without this, a single noisy reading would draw an enormous arrow across the scene.

## 2. Data curriculum

**100 demonstrations per task**, teleoperated via **Oculus Quest 2**, recorded at ~10 Hz, varying spatial arrangement and lighting.

Four real tasks, chosen so that the decisive variable is **invisible**:
- **medicine** — is the bottle full or empty? (mass)
- **balance** — where is the centre of mass? ("far" vs "near")
- **gear** — insertion requiring force reasoning for alignment
- **plug** — cordless plug end from the IndustReal benchmark into a tabletop socket

## 3. Model

**π₀.₅**, unmodified. Inputs: shear-annotated over-the-shoulder and wrist images, 7-dim joint state, task language. Outputs: 8-dim delta joint angles + binary gripper. Action horizon **H = 10**. AdamW, 20k steps, batch 32.

## 4. Baselines — the controls that make the result meaningful

All trained with identical optimiser, schedule, dataset and steps:

- **π₀.₅** — vision-only fine-tuning
- **π₀.₅ + Tactile** — raw GelSight images as additional "viewpoints"
- **π₀.₅ + Encoder** — the standard decoupled approach: a from-scratch 4-layer stride-2 CNN (channels 16/32/64/128, 3×3 kernels, GELU, per-layer LayerNorm), each GelSight frame resized to 64×64 → 4×4×128 → 16 patch tokens projected to LLM width 2048, weights shared between left and right sensors, trained jointly

That third baseline is exactly what most of the VTLA literature does, specified in enough detail to reproduce.

## 5. Results

**30 trials per task. TAP-VLA: 94/120 = 78%**, leading on every individual task, against **under 50%** for all three baselines.

| Task | Baselines | TAP-VLA |
|---|---|---|
| medicine | at or below **chance (50%)** | **24/30** |
| balance | at or below **chance (1/3)** | **25/30** |
| gear | some corrective behaviour (tactile baselines); π₀.₅ wedges/jams | best |
| plug | same | best |

Two findings, both important:

**(1) The three baselines are indistinguishable from each other.** Raw-tactile-as-viewpoint, learned encoder, and vision-only all land in the same band. The authors' conclusion is blunt: *"adding tactile data alone is insufficient; how it is injected into the policy is what dictates performance."*

**(2) On medicine and balance the baselines are at chance.** None of them — including the encoder baseline with full access to raw GelSight images — reliably identifies the bottle's mass or the object's centre of mass. The signal is present and they cannot use it. TAP-VLA reads it off the rendered arrows.

The qualitative rollouts show what the annotations encode: whether the medicine bottle is full or empty (guiding bin selection), the implied centre of mass (guiding placement on balance), and continuously changing shears driving corrective alignment during insertion.

## 6. Honest limitations, well stated

- **Information loss** — shear extraction and down-sampling discard tactile information, most directly *"the detailed grasp geometry conveyed by the raw GelSight image."*
- **Occlusion** — annotations can cover task-relevant visual content in cluttered scenes; mitigable by also providing an unannotated view, at the cost of redundancy.
- **Scaling** — a multi-fingered hand with many sensors would produce a denser overlay that "likely reduces interpretability."
- The authors position TAP-VLA as **complementary** to large-scale tactile pre-training: a way to use existing VLAs today while native tactile architectures mature.

## 7. What it adds that the others don't

The **strongest evidence in this survey that presentation dominates signal**. The encoder baseline has the same GelSight images, the same backbone, the same data and the same compute, and performs at chance on tasks TAP-VLA solves 80% of the time. That result reframes the whole tactile-VLA design space: the constraint may not be tactile representation quality ([[anytouch2]], [[ftp-1]]) or fusion architecture ([[at-vla]], [[vitar]]) but simply **whether the signal arrives in a form the pretrained model already knows how to read.**

The generalisation the authors propose is worth taking seriously: *any* signal admitting a 2D rendering — other tactile representations, force/torque time series, audio spectrograms, depth overlays — could be supplied to a pretrained VLA the same way. Compare [[rgb-s]], which independently arrives at rendering contact into the image plane, from the saliency direction rather than the prompting one.
