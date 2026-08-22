# CAAT — Contact-Aware Attention Scaling and Tactile Masking for Data-Efficient Contact-Rich Manipulation

**arXiv:2608.01102** · ShanghaiTech + Beihang + BIGAI + Zhejiang + BUPT (J. Jiang, Y. Huang, H. Liang, P. Lin, S. Luo, Dong, J. Wu, Xiao, W. Li, Jiao) · Aug 2026 · [site](https://mrjiangjm.github.io/caat/)

**One line.** A **lightweight, architecture-agnostic contact prior** — scale attention toward vision before contact and toward tactile during it, and mask static tactile background — that drops into ACT, Diffusion Policy or π₀ without touching their action decoders.

## 1. The argument

*"During much of a manipulation trajectory, the robot operates in free space or performs coarse alignment, for which visual observations provide sufficient global information... By contrast, tactile sensing is most informative during contact phases."* Therefore *"although tactile sensing is indispensable for dexterous manipulation, its utility is inherently **stage-dependent**."*

But standard designs — **token concatenation** and **learnable gating** — *"lack explicit contact-aware priors, making it difficult to efficiently learn effective cross-modal representations from demonstrations."* The efficiency framing is the point: a learnable gate *can* discover the contact prior, but it has to spend demonstrations doing so.

## 2. Method — two mechanisms, both simple

**(a) Contact-aware attention scaling** — emphasise vision tokens before contact and tactile tokens during contact. A contact predictor compares the current tactile image against a **non-contact reference image** (the first frame of each episode) and outputs a probability in [0,1], thresholded at **0.5**.

**(b) Dynamic Tactile Masking (DTM)** — suppress **static background tokens** by comparing current tactile against the non-contact reference, with a **cosine-similarity threshold ρ = 0.8**. Tokens too similar to the no-contact baseline are masked out.

Both are computed, not learned, and neither modifies the action decoder — so CAAT is a drop-in for any Transformer policy.

## 3. Results

**Simulation** (UniVTAC, 5 tasks, 100 rollouts × 100 seeds): CAAT + ACT improves average success by **+18.0 points over direct visuo-tactile fusion** and **+10.0 over gated fusion**.

**Real world** (visuo-tactile UMI platform, 150 demos/task, 20 trials/task): **60.0% average across ACT, Diffusion Policy and π₀**, beating the strongest baseline by **+21.1 points on average** — the cross-architecture consistency being the claim.

**Data efficiency** (average success % across five UniVTAC tasks, by demonstration budget):

| Method | small | → | → | large |
|---|---|---|---|---|
| ACT + DTM | 21.6 | 25.4 | 22.2 | 51.6 |
| Gated ACT + DTM | 15.4 | 30.0 | 27.8 | 59.6 |
| **CAAT (Numerical)** | **24.8** | **34.0** | **38.4** | **69.6** |

CAAT leads at every budget, and the margin over gated fusion **widens** in the mid-data regime (38.4 vs 27.8) — exactly what an injected prior should do: it substitutes for the demonstrations a learnable gate would need in order to discover the same structure. Note also that gated fusion is *worse* than plain concatenation at the smallest budget (15.4 vs 21.6), because the gate itself must be learned.

## 4. What it adds that the others don't

**A contact prior that costs nothing and ports anywhere.** Every gating design in this survey learns its gate ([[feelworld]], [[omnivta]], [[at-vla]]) or computes it from frame difference ([[dream-tac]]); CAAT computes it against a per-episode non-contact reference — the same baseline-subtraction idea [[n0-vtla]] uses for encoding — and adds **masking of static tactile background**, which nothing else does. The data-efficiency table is the strongest evidence in the survey that **explicit priors beat learned gates when demonstrations are scarce**, which is the regime nearly all tactile robot learning operates in.
