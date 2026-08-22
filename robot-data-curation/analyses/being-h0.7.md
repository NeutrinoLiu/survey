# Being-H0.7 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Being-H0.7: A Latent World-Action Model from Egocentric Videos |
| **Org** | BeingBeyond |
| **Date** | 2026-05 (arXiv 2605.00078) |
| **Artifact** | `paper.pdf`, `paper.html`; research.beingbeyond.com/being-h07 |
| **Disclosure level** | **A — full paper** |
| **Corpus** | **200,000 hours** egocentric human video + **15,000 hours** robot demonstrations |
| **Stance** | **Avoid pixel prediction; compress the future into latents.** The world-modeling signal is extracted without paying the cost of video rollout. |
| **Lineage** | Being-H0 → Being-H0.5 → **Being-H0.7** (latest in stream) |

## 1. Corpus scale and position

At **200K hours of human video + 15K hours of robot data**, Being-H0.7 sits between EgoScale (20.8K h) and DYNA-2 (1M h) on the human-video axis, but unlike DYNA-2 it retains a substantial robot-data component (a **13:1** human:robot ratio).

Stream evolution, as the paper describes it: Being-H0 *"scal[ed] this paradigm through **motion-tokenized reconstructed human hand trajectories**"*; Being-H0.5 *"further generaliz[ed] this direction toward unified cross-embodiment"* learning; Being-H0.7 replaces explicit motion tokens with a **latent predictive interface**.

## 2. The architectural choice that shapes the data requirement

> *"Instead of predicting future frames, Being-H0.7 introduces a small set of **learnable latent queries** between the multimodal context and the action tokens."*

**Dual-branch, future-informed training:**
| Branch | Input | Role |
|---|---|---|
| **Prior** (deployable) | Current multimodal context only | Infers action-useful predictive factors |
| **Posterior** (training only) | Latent queries **replaced by embeddings extracted from subsequent/future observations** | Supervises the prior via joint latent alignment |

> *"Jointly aligning the two branches at the latent reasoning space leads the prior branch to reason **future-aware, action-useful structure from current observations alone**. At inference, Being-H0.7 **discards the posterior branch and performs no visual rollout**."*

**Data-pipeline consequence:** the training signal is *"future observations"*, i.e. later frames of the same clip. This means:
- **No future-frame pixel reconstruction target** is needed → no requirement that video be visually clean enough to *generate*, only clean enough to *encode*. Compression/blur artifacts that would poison a video-diffusion objective are largely tolerable here.
- **No action labels are needed for the world-modeling signal** — any temporally coherent egocentric clip contributes.
- The binding data constraint is **temporal coherence**, not annotation quality. This is why a 200K-hour corpus is tractable.

The paper names the problem this solves: *"action-relevant cues are not explicitly annotated"* in human video, so the latent queries are trained to *discover* them rather than to match a supplied label — sidestepping the pseudo-action-quality problem that ACE-Ego-0 addresses with reliability weighting and DYNA-2 addresses with a hand-pose quality bar.

## 3. Where it sits among the three human-video routes

| Route | How human video is consumed | Data-quality burden |
|---|---|---|
| **Pseudo-action labels** (EgoScale, ACE-Ego-0, Qwen-RobotManip H2R, DYNA-2 action tier) | Hand pose → robot action | **High** — reconstruction errors become learned artifacts |
| **Video prediction** (DYNA-2 video tier, LingBot-VA, Hydra-0) | Future-frame targets | **Medium** — needs visually generable video |
| **Latent world-action** (**Being-H0.7**) | Future *embeddings* as latent targets | **Low** — needs only temporal coherence |

This ordering is a useful lens for the whole survey: **the choice of objective determines how expensive your cleaning pipeline has to be.** Being-H0.7 is the cheapest-to-curate point on that spectrum, which is what makes 200K hours feasible for a research group.

## 4. Results
Across **six widely used simulation benchmarks** (including CALVIN ABCD→D and ABC→D, LIBERO, LIBERO-Plus), Being-H0.7 *"reaches the strongest overall performance and the best average rank"*, plus diverse real-world tasks. Evaluated in two configurations — trained on standard LIBERO, and fine-tuned on the augmented LIBERO-Plus dataset — with explicit **zero-shot generalization under controlled environmental perturbations**.

## 5. What they do not do
- The cleaning/ingestion pipeline for the 200K-hour corpus is not detailed in the report — source composition, filtering, and dedup are not enumerated (contrast HumanNet or ACE-Ego-0).
- No fitted data-scaling curve at the 200K-hour scale.
- No published corpus release.
- The latent objective's tolerance to noisy video is an inference from the design, not a measured ablation.

## 6. Transferable takeaways
1. **Choose the objective that minimizes your curation bill.** Latent-future alignment needs neither action labels nor generable pixels — only temporal coherence — which is why the corpus can be an order of magnitude larger than action-labeled alternatives.
2. **Discard the training-only branch at inference.** The posterior branch buys world-model supervision without paying rollout latency in deployment.
3. **Let the model discover action-relevant structure** rather than supplying a noisy pseudo-label for it — an alternative to reliability weighting (ACE-Ego-0) and quality gating (DYNA-2).
4. Read against **Ψ₀** (800 h, single clean source) and **DYNA-2** (1M h, pseudo-actions): Being-H0.7's 200K h sits in between and gets there by changing what the data must *support*, not by cleaning harder.
