# AT-VLA — Adaptive Tactile Injection for Enhanced Feedback Reaction in Vision-Language-Action Models

**arXiv:2605.07308** (v2) · **CVPR 2026** · Peking University + PrimeBot + PKU Lab (X. Li, Cai, J. Xu, Zhu, Fan, Shen, Ren, Dong) · May 2026 · [site](https://sites.google.com/view/at-vla)

**One line.** Contains the single most useful ablation in the tactile-VLA literature: **three tactile formats × two injection strategies**, showing that *every* format hurts when injected directly and that **lower-dimensional tactile is better**.

## 1. What "tactile" means here — three formats, compared head to head

| Format | Representation | Encoder |
|---|---|---|
| **Visual-tactile image** | contact geometry as `H×W×3` RGB | pretrained **Sparsh** |
| **Marker 2D** | surface-marker displacement, `N×2` | lightweight MLP + nonlinearity |
| **Force 6D** | 3D tangential + 3D normal force vector | — |

Testing all three under one architecture is what makes the results interpretable.

## 2. The two problems

**(a) Modality introduction disrupts pretraining.** Open-source manipulation pretraining data rarely contains tactile, so tactile is added at downstream finetuning — and *"the direct introduction of these new modalities may disrupt the existing pretrained knowledge, such as visual perception or object grounding."*

**(b) VLA inference is too slow for tactile.** High-frequency touch demands rapid adjustment; a slow VLA cannot close the loop.

## 3. Model

Base: **GO-1**. Two mechanisms:

**Adaptive Tactile Injection.** A learnable **Tactile Gate** (score > 0.5, trained with an explicit **gate loss**) decides whether tactile features enter the action expert. Paired with **Adaptive Cross Attention**, which *conditionally switches the query source*: when the gate is on, attention queries come from tactile tokens; when off, they are identical to the vanilla VLA. Critically, this *"integrates tactile information when the gate is activated, without the need to modify the model structure or feature dimensionality compared to the inactive state"* — the non-contact computation path is bit-for-bit the pretrained one.

**Tactile Reaction Dual-Stream.**
- **Slow stream** — vision + language through the large VLM at low frequency, preserving pretrained perception and localisation.
- **Fast stream** — tactile fed continuously into the action expert at high frequency.

Each action prediction conditions on the **latest tactile feedback** and the **most recent vision-language output falling within the same action-chunk horizon**. Closed-loop response in **0.04 s**.

## 4. Experiment setup

Four real contact-rich tasks: **Unzip Bag, Wipe Vase, Unscrew Lid, Stamp**. Seven ablation configurations crossing components against tactile formats.

## 5. Results — the ablation table that matters

| Ex | Gate | Adapt. XAttn | Direct incorp. | Dual-stream | Format | Unzip | Wipe | Unscrew | Stamp | **AVG** |
|---|---|---|---|---|---|---|---|---|---|---|
| Ex0 | – | – | – | – | (none, vanilla GO-1) | 0.20 | 0.33 | 0.07 | 0.27 | **0.22** |
| Ex1 | – | ✓ | ✓ | – | Force 6D | 0.07 | 0.13 | 0.07 | 0.20 | **0.13** |
| Ex2 | ✓ | ✓ | – | – | Force 6D | 0.27 | 0.40 | 0.53 | 0.33 | **0.39** |
| **Ex3 (ours)** | ✓ | ✓ | – | ✓ | Force 6D | 0.33 | 0.46 | 0.67 | 0.53 | **0.50** |
| Ex4 | – | ✓ | ✓ | – | Marker 2D | 0.00 | 0.13 | 0.07 | 0.00 | **0.05** |
| Ex5 | ✓ | ✓ | – | ✓ | Marker 2D | 0.27 | 0.33 | 0.27 | 0.40 | **0.32** |
| Ex6 | – | ✓ | ✓ | – | V-T image | 0.00 | 0.00 | 0.07 | 0.00 | **0.02** |
| Ex7 | ✓ | ✓ | – | ✓ | V-T image | 0.27 | 0.46 | 0.47 | 0.40 | **0.40** |

Three findings, each sharp:

**(1) Direct tactile injection is catastrophic, and worse the higher-dimensional the tactile.** Against the 0.22 vanilla baseline: Force 6D → **0.13**, Marker 2D → **0.05**, V-T image → **0.02**. The failure mode is diagnosed precisely: *"Most failures occur during object grasping, indicating degraded visual grounding and perception ability."* The model does not fail at contact — it fails at **reaching**, because tactile tokens have displaced the pretrained visual grounding.

**(2) Gating recovers all of it, and then some.** +17 points over vanilla from the gate alone (Ex0→Ex2), then +11 more from the dual-stream (Ex2→Ex3), for 0.22 → **0.50**. Ex5 beats Ex4 by 27 points and Ex7 beats Ex6 by **38 points** — the framework works across all three formats.

**(3) Lower-dimensional tactile wins.** Force 6D (0.50) > V-T image (0.40) > Marker 2D (0.32). The hypothesis: *"higher-dimensional tactile inputs may excessively perturb the pretrained representation space, as they introduce a larger number of tactile tokens."* This is a direct, measured counterweight to the assumption running through the world-model cluster that richer tactile representations are better — at least when grafting onto a pretrained VLA, the opposite holds.

The dual-stream's value is illustrated concretely on Unzip, *"where delayed reactions may cause the zipper to jam."*

**Modality-agnostic robustness.** AT-VLA *"maintains strong performance even in the absence of tactile signals during inference"*, comparable to the vanilla VLA — the adaptive training strategy apparently lets it infer approximate tactile cues from visual features at test time. The same phenomenon [[unitacvla]] reports for its sensor-free variant, here framed as a deployment virtue for settings where tactile is unstable or unavailable.

## 6. What it adds that the others don't

The **format comparison under a fixed architecture**. Nothing else in this survey answers "should I use force, markers, or gel images?" empirically, and the answer — fewer dimensions, when attaching to a pretrained VLA — cuts against most of the field. Combined with the Ex1/Ex4/Ex6 collapse, it gives the strongest quantitative evidence for the claim that [[restacvla]], [[vitar]] and [[tactile-wam]] each make qualitatively: **naive tactile fusion is not neutral, it is actively destructive to pretrained visual grounding.**
