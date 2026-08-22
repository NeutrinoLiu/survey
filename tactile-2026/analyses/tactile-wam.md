# Tactile-WAM — Touch-Aware World Action Model with Tactile Asymmetric Attention

**arXiv:2606.26663** (v2, Aug 2026) · Ant Group + Institute of Automation CAS + HKU + Zhejiang University (S. Wu, You, Zhu, Y. Liu, … Zhao) · Jun 2026

**One line.** Names and *measures* **tactile pollution** — the degradation of a visually pretrained world model when you fine-tune it with a much smaller tactile corpus — and separately shows that **tactile pixel change does not correlate with contact change**, which invalidates pixel reconstruction as a tactile prediction objective.

## 1. What "tactile" means here

Vision-based tactile images from left and right sensors, `o^{τ,r}_t ∈ R^{H_τ×W_τ×3}`, `r ∈ {L,R}` — horizontally mosaicked and encoded by the same frozen **Wan2.2 VAE** as RGB.

But the paper's real tactile representation is a **six-dimensional touch-aware proxy derived from optical flow between consecutive tactile images**, because of its second key observation (below). The proxy is used twice: observed proxy changes drive an attention bias, and future-proxy supervision constrains the predicted tactile latents.

## 2. Two key observations, both measured

**(a) Tactile pollution.** In a DiT, all modality tokens are concatenated, so visual queries attend over `K = [K^v, K^τ, K^a, K^s]`. On UniVTAC (**800 training trajectories**), this "Naive VT-WAM" shows *higher training loss* than the RGB-only DreamZero baseline, **50% higher MSE** on future-frame prediction, and visible blur around manipulated objects. Fine-tuning a visually pretrained WAM with limited tactile data **interferes with the visual dynamics prior**.

**(b) Pixel–contact misalignment.** Sampling **5,000 tactile image pairs** and plotting pixel variation against contact variation yields **no clear linear correlation**. Large pixel changes with limited contact variation; large contact changes with small pixel differences; pairs with identical pixel variation differing substantially in contact change. Therefore *"pixel reconstruction objectives alone cannot ensure predicted tactile representations preserve contact changes critical for action generation."*

This is the most consequential measurement in the 2026 tactile world-model literature, because a large fraction of these papers optimise exactly the objective it invalidates.

## 3. Model

Wan2.2-based. RGB and tactile histories encoded independently by the frozen VAE with modality embeddings; a language-conditioned **Wan DiT** jointly predicts future video latents, future tactile latents, and action chunks under conditional flow matching.

Objective: `L = L_video + L_action + λ_τ L^τ_FM + L_F`, where `L_F` is the virtual-force (proxy) supervision.

## 4. How tactile enters the model — TAAM

**Tactile Asymmetric Attention Mechanism**, two halves:

- **VideoClean mask** — video queries are **blocked from tactile keys**. Tactile can read vision; vision cannot read tactile. This directly targets pollution.
- **Contact-change-aware attention bias** — computed from observed **proxy** changes (not pixel changes), it strengthens *action* queries' attention to tactile keys during contact transitions, within each causal block.

Plus **future-proxy supervision** so predicted tactile latents preserve action-relevant contact dynamics rather than appearance.

Note this is the same structural conclusion as [[hitac-wam]] (one-way isolation) and [[feelworld]] (contact-gated), reached from a third direction — and directly opposed to [[n0-twam]]'s "vision and touch stay fully attentive to one another".

## 5. Experiment setup

Nine simulation tasks (UniVTAC, ManiFeel) and five real-robot tasks across six conditions, 20 trials each. Baselines: DreamZero (RGB-only WAM), Naive VT-WAM, π₀.₅, VT-WAM.

The pollution evaluation deserves note for methodology: **84 paired held-out UniVTAC samples at step-matched 20K checkpoints**, identical conditioning frames and diffusion seeds, with the 20K DreamZero checkpoint as reference. The measured quantity is *prediction-to-prediction distance* — how much tactile conditioning perturbs the learned visual trajectory — not distance to ground truth. Bootstrap confidence intervals throughout.

## 6. Does it work?

**Does VideoClean actually protect the visual prior?**

| Metric | w/o VideoClean | VideoClean | Change | Paired improvement [95% CI] | VideoClean closer |
|---|---|---|---|---|---|
| MSE ↓ | 0.001568 | 0.001227 | **−21.8%** | +0.000342 [0.000117, 0.000555] | 84.5% |
| MAE ↓ | 0.020904 | 0.017190 | −17.8% | +0.003714 [0.002877, 0.004591] | 90.5% |
| PSNR ↑ | 31.016 | 32.020 | +1.005 dB | +1.005 [0.646, 1.356] | 81.0% |

All CIs exclude zero. And critically, on **ground-truth** prediction quality **all CIs include zero** — so VideoClean does not make video prediction better or worse against reality, it just stops tactile from *dragging the trajectory away from the pretrained prior*.

The horizon breakdown shows the mechanism is drift suppression:

| Chunk | Error reduction | VideoClean closer |
|---|---|---|
| H1 | −1.7% | 13.1% |
| H2 | +3.3% | 70.2% |
| H3 | +22.4% | 88.1% |
| H4 | **+38.5%** | **90.5%** |

At one step ahead, unrestricted fusion is fine. Pollution is a **compounding** phenomenon — which is exactly why single-step prediction metrics would miss it entirely.

**Cumulative ablation on ManiFeel (60K steps):**

| Method | Future touch | VideoClean | Force | Bias | Success |
|---|---|---|---|---|---|
| RGB-only WAM (DreamZero) | – | – | – | – | 0.156 |
| Naive VT-WAM | ✓ | – | – | – | **0.151** |
| + VideoClean | ✓ | ✓ | – | – | 0.284 |
| **Full Tactile-WAM** | ✓ | ✓ | ✓ | ✓ | **0.327** |

The second row is the finding: **naive tactile injection is slightly worse than no tactile at all** (0.151 vs 0.156). Adding VideoClean alone buys **+13.3 points** — the largest single ablation gain — leading to the authors' conclusion that *"visual–tactile interaction is a greater bottleneck than tactile availability alone."* Force supervision and the bias add a further +4.3 together.

**Overall:** ManiFeel 147/450 = **32.7%** vs DreamZero 15.6% (2.10×). π₀.₅ attains a higher *overall* rate (36.7%) but Tactile-WAM leads on the four contact-dominated tasks (peg, gear, bulb, bolt-nut). On UniVTAC among from-scratch models: 30.0% vs DreamZero 21.3%, with tube insertion 17/20 versus 4/20.

**Real robot:** 59/120 = **49.2%**, versus π₀.₅ 24.2%, DreamZero 21.7%, VT-WAM 15.8%. First in **all six conditions**, including **dim-light power insertion, where it retains 50% while DreamZero falls from 55% to 20%** — the clearest demonstration in this survey that touch is what remains when vision degrades.

**Honest identification limits, stated by the authors.** Force supervision and attention bias are introduced together, so the ablation measures only their **joint** contribution; force-only and bias-only variants are needed. And the bulb-insertion attention visualisation (bias redirects action attention to tactile after initial contact, weakening as contact stabilises; without it residual contact develops into slip) "does not isolate its success-rate contribution" absent a bias-only real-robot variant.

## 7. What it adds that the others don't

Two measurements that should change how the field builds and evaluates these models: **pollution compounds over horizon** (so report H4, not H1), and **tactile pixel similarity ≠ contact similarity** (so do not reconstruct tactile pixels). Its statistical treatment — paired samples, matched seeds, bootstrap CIs, and a null result reported as a null result — is the most careful in the world-model cluster. Read directly against [[n0-twam]], whose scale may be the reason it can afford the fusion Tactile-WAM shows is harmful at 800 trajectories.
