# VT-WAM — Visual-Tactile World Action Model for Contact-Rich Manipulation

**arXiv:2607.02503** · SKL-MAIS Institute of Automation CAS + UCAS + TARS Robotics + NUS + Fudan (Tian, Y. Zheng, Y. Zheng, Gu, Zang, Qin, Li, H. Li, Ding, Zhao) · Jul 2026 · [site](https://vt-wam.github.io/)

**One line.** Measures the **effective contact ratio** across six real tasks — touch is informative only 26–41% of the time — and designs both its attention mask and an auxiliary loss around that single number.

## 1. What "tactile" means here

**3D deformation fields from two tactile surfaces**: `O_t ∈ R^{T_t×6×H_t×W_t}` — six channels for two contact surfaces, i.e. a 3-vector displacement field per surface.

The paper's most useful empirical contribution is quantifying the problem everyone else asserts. **Effective contact ratio** per task:

| Task | Ratio |
|---|---|
| Wipe Board | 0.41 |
| Wipe Vase | 0.34 |
| Peel Cucumber | 0.32 |
| Insert Plug | 0.29 |
| Swipe Card | 0.31 |
| Insert Tube | 0.26 |

So **59–74% of frames carry no informative tactile signal**. The consequence the authors draw: this temporal imbalance in information availability *causes neural networks to favour visual evidence during joint training*, leaving tactile underutilised. Not a fusion-architecture failure — a **gradient-statistics** failure. Both of VT-WAM's contributions attack it.

## 2. Data curriculum

Six real-world tasks in two regimes: **surface-interaction** (Wipe Board, Wipe Vase, Peel Cucumber, Swipe Card) and **constrained insertion** (Insert Plug, Insert Tube). Wrist camera only, plus proprioceptive state and language.

## 3. Model

Three modality experts under a unified **flow-matching** objective (joint future-visual, future-tactile-deformation, and action prediction):

- **Visual expert** — wrist sequence `O_v ∈ R^{T_v×3×H×W}` through the **Wan2.2 video VAE**, patchified to `X_v ∈ R^{N_v×d}`
- **Tactile expert** — deformation sequence through a **pretrained tactile VAE** (following OmniVTA) to `X_t ∈ R^{N_t×d}`
- **Action expert** — chunk `A ∈ R^{S_a×D_a}` linearly projected to `X_a ∈ R^{S_a×d}`

Language and proprioception reach each expert by cross-attention. Q/K/V are concatenated `[visual ; tactile ; action]` and run through masked attention.

## 4. How tactile enters the model — two mechanisms

**(a) Asymmetric MoT Attention.** Action tokens are routed to a **first-frame visual anchor** for scene context and to the **full tactile sequence** for contact evolution. The masks differ between training and inference: at inference only `V_0` (the anchor frame) is kept on the visual side while the entire tactile sequence `T_0…T_n` remains attendable.

The engineering payoff is specific: it enables **visual-cache inference** — the expensive video branch collapses to one cached anchor frame — **without discarding the tactile dynamics** that contact phases need. Efficiency is bought from vision rather than from touch, which inverts the usual trade in efficient world-action models (Fast-WAM, Metis, Efficient-WAM all shorten or bypass the predicted future).

**(b) Contact-gated AVTAG** (Action–Visual–Tactile Attention Guidance). A **training-only hinge ranking loss** on the attention map itself:

```
L_AVTAG = max(0,  p_v − p_t)     applied only during contact phases
```

where `p_v` and `p_t` are the action queries' attention mass on visual vs. tactile keys, computed with **stop-gradient on the keys** `Q_A · sg[K_V, K_T]^T`. It directly *penalises the model for attending more to vision than to touch while contact is happening*, and does nothing outside contact.

This is a distinctive move. Rather than restructuring the architecture to protect or promote a modality ([[feelworld]]'s gate, [[hitac-wam]]'s directed mask, [[n0-twam]]'s separate experts), VT-WAM leaves the architecture symmetric and **supervises the attention distribution**. Because it is training-only, inference-time architecture is unchanged.

## 5. Experiment setup

Six real tasks, two regimes. Baselines: **Fast-WAM** (vision-only world action model), **OmniVTLA** (vision-tactile-language-action), **RDP** (Reactive Diffusion Policy).

## 6. Does it work?

| Method | Surface-interaction | Constrained insertion | Average |
|---|---|---|---|
| RDP | 33.33 | 20.00 | — |
| OmniVTLA | 48.33 | 33.33 | 35.83 |
| Fast-WAM | 56.67 | 33.33 | 45.00 |
| **VT-WAM** | **81.67** | **61.67** | **71.67** |

**+26.67** over Fast-WAM and **+35.84** over OmniVTLA, with gains in both regimes (+25.00 surface, +28.34 insertion). Notably OmniVTLA — which *does* consume tactile — underperforms the vision-only Fast-WAM on average, which is exactly the "tactile available but not exploited" pathology the effective-contact-ratio analysis predicts, and the same phenomenon [[softvtbench]] documents from the benchmark side.

Ablations confirm both designs contribute: modelling tactile deformation *dynamics* (rather than feeding current tactile) and guiding contact-phase tactile attention are each necessary.

## 7. What it adds that the others don't

The **effective contact ratio** as a measured quantity, and AVTAG as the only attempt in this survey to fix modality imbalance with a **loss on the attention map** rather than a structural constraint — which means it costs nothing at inference. The asymmetric mask that caches vision while preserving full tactile history is also the cleanest efficiency argument here: in contact-rich settings the *visual* future is the redundant part.
