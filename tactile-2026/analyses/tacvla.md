# TacVLA — Contact-Aware Tactile Fusion for Robust Vision-Language-Action Manipulation

**arXiv:2603.12665** (v2) · Purdue University + IIT Genova + Università di Genova (K. Zhang, H. Zhang, Xu, Z. Zhang, Prince, X. Li, Han, Y. Zhou, Ajoudani, She) · Mar 2026 · [site](https://sites.google.com/view/tacvla)

**One line.** The token-budget argument: represent tactile as **low-dimensional tokens rather than images**, and switch them on only when contact is detected — cheap fusion that survives camera occlusion.

## 1. What "tactile" means here

A **compact tactile array**, tokenised as a **low-dimensional tactile map (H×W)** rather than an image. The reasoning is explicitly computational: most tactile-VLA work treats tactile as image-like input, *"relying on dense pixel representations that increase token length and computational cost in transformer architectures."*

Hardware: 7-DoF Franka with tactile sensors, front camera and wrist camera.

## 2. Model

Standard VLA skeleton with modality-specific front-ends:
- **SigLIP** vision encoder (front + wrist RGB)
- Language tokenizer
- **Tactile encoder** → MLP → tactile tokens
- **Gemma 2.6B** VLM backbone
- Action expert → actions

Fused visual, language and tactile tokens are jointly processed in the transformer.

## 3. How tactile enters the model — contact-aware gating

A **binary tactile mask** derived from detected contact state multiplies the tactile tokens along the temporal timeline:

```
tactile tokens:  … 1.4  0.2  3.1  6.5  2.3 …
tactile mask:    …  0    0    1    1    1  …
```

Tactile tokens are **selectively activated only when contact is detected**, so during non-contact phases they contribute nothing — no cross-modal interference, no wasted attention.

This is the simplest gate in the survey: a hard binary mask on tokens, versus [[dream-tac]]'s continuous frame-difference bias, [[feelworld]]'s learned contact probability gating cross-attention, and [[restacvla]]'s visual-uncertainty gate. Its virtue is that it costs nothing and needs no learned components.

## 4. Experiment setup

Task selection is the paper's other contribution — it targets a regime the field under-tests:

**Four constraint-locked disassembly tasks**, each with a different geometric constraint:
1. **Tight shaft** — positioning → close → adjust slightly → pull up → place
2. **Press clip** — positioning → close → press the clip → pull up → place
3. **Shaft rotation** — positioning → close → rotate 90° → pull up → place
4. **Slide pull** — positioning → close → slide inward → pull up → place

Plus **in-box picking** (move down → explore → grasp → move up → place), where the box walls and the gripper occlude the view.

Robustness tests: **blocked camera** and **human disturbance recovery**.

The authors' framing is that existing tactile-VLA evaluation has "limited emphasis on contact-intensive fine-grained manipulation tasks that require sustained and precise contact control, such as disassembly" — and disassembly is a good choice precisely because the constraint is invisible: you cannot see whether a clip has released.

## 5. Does it work?

| Setting | Improvement over baselines |
|---|---|
| Constraint-locked disassembly | **+20%** average success |
| In-box picking | **+60%** |
| Visual occlusion | **2.1×** |

The ordering is the informative part. The gain is smallest on disassembly (+20 points), large on in-box picking (+60), and largest under **deliberate occlusion (2.1×)**. Touch buys the most exactly where vision is unavailable rather than merely insufficient — consistent with [[tactile-wam]]'s dim-light insertion result and [[n0-twam]]'s visual-perturbation axis, and a reminder that headline tactile gains are heavily task-selection-dependent.

## 6. What it adds that the others don't

Two practical points rather than a conceptual one. **(a)** The **token-budget argument**: dense tactile images are an expensive way to carry a sparse signal, and a compact array tokenisation gives comparable benefit at a fraction of the sequence length — a consideration that becomes acute for the 100 Hz streams [[tacmamba]] handles and the 12-slot budgets [[disentvtf]] studies. **(b)** **Constraint-locked disassembly** as a task family, where the thing the policy must detect (a clip releasing, a shaft freeing) has essentially no visual signature. Most contact-rich benchmarks here are insertion and wiping; disassembly is a harder and under-used probe.
