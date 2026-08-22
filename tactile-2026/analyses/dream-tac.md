# Dream-Tac — A Unified Tactile World Action Model for Contact-Rich Robot Manipulation

**arXiv:2606.08737** · Peking University + HKUST + Nanjing University (Lou, Ye, Fu, Cen, Chi, Lyu, Jia, Han, Lu, S. Zhang) · Jun 2026 · [code](https://github.com/LYFCLOUDFAN/Dream-Tac)

**One line.** Gates tactile influence by a **non-learned** scalar computed from consecutive tactile frame difference — then publishes a trace of that gate over real episodes, which is the only direct evidence in this survey of *when* a tactile gate actually fires.

## 1. What "tactile" means here

Tactile RGB from a two-view (left/right) sensor, encoded by the same **pretrained Wan VAE** as vision. The framing argument: RGB gives no clear indication of contact, while tactile variation reveals subtle interaction cues.

The observation that drives the design: **tactile signals are sparse and transient** — long periods of stasis punctuated by brief critical events (contact onset, slip, release). *"Treating tactile tokens symmetrically with other modalities risks diluting the precise interaction cues most essential for successful manipulation."*

A nice sanity check: a t-SNE of tactile representations from the frozen Wan VAE shows **distinct clusters per manipulation action** (cut / wipe / peel / insert / static), so a general-purpose video VAE already encodes action-discriminative tactile structure without tactile-specific pretraining.

## 2. Data curriculum

Six real-world contact-rich tasks: **Peel Cucumber, Cut Banana, Insert USB, Play Mahjong, Pick and Place, Clean Whiteboard**. Plus four out-of-distribution generalisation settings — table-height variation (peel cucumber), spatial variation (pick baguette), object variation (play mahjong: trained on Red Dragon / One Dot, tested on other suits), background variation (cut banana).

## 3. Model

Built on a **pretrained video generative backbone** (Cosmos lineage), jointly predicting future visual observations, future tactile observations, and an action chunk `a_{1:H}` conditioned on current visual `o`, tactile `x`, and instruction `l`.

## 4. How tactile enters the model — the contact gate

**The gate is not learned.** It is computed directly from the data:

1. Tactile frames are normalised by 255; `ρ_t` = mean absolute difference between consecutive tactile frames, taken as the **max of the left and right sensor values**; `ρ_0 = 0`.
2. `ρ_t` is mapped to a gate `g_t ∈ [g_min, g_max]`.
3. `g_t` scales a **contact-aware attention bias** added to the attention logits, so non-tactile queries (action, vision, state) are biased toward tactile keys in proportion to how fast contact is changing.

Crucially, the authors state the limit of what the gate does: *"the fine-grained allocation of attention across tactile tokens remains governed by the content term qᵀk; g_t only modulates, at a coarse level, how strongly non-tactile queries are biased toward tactile keys."* It is a volume knob, not a router.

**Published gate statistics** (874 timesteps from five sampled Peel Cucumber episodes): `ρ_t` median **1.73×10⁻³**, 90th percentile **6.08×10⁻³**, mean 2.73×10⁻³, std 2.32×10⁻³, **coefficient of variation ≈ 0.85**. Within each episode `g_t` spans about **0.85 of the [g_min, g_max] interval**, while the episode time-average ranges 0.48–0.61.

**Published gate traces** — the most useful figure. Across four of five episodes the temporal structure is consistent: an early **roughly periodic oscillation** in `g_t` during the approach phase (attributed to low-level tactile sensor noise and minor non-contact disturbances), then a rapid rise to a **sustained high regime once contact occurs**, then a return to low after the interaction subsides. The fifth episode starts already high — traced to the robot's initial pose not being strictly fixed across demonstrations, so that trajectory begins near contact.

That pre-contact oscillation is worth noting: it is exactly the free-space sensor noise that [[feelworld]] and [[tactile-wam]] argue must be suppressed, and here it does modulate attention, just weakly.

## 5. Acceleration — a dual-level system

Adding tactile sequences to a Diffusion Transformer compounds quadratic self-attention with iterative denoising. Two fixes:

- **Training** — the gated-bias attention is re-implemented in a **FlashAttention-compatible** formulation that preserves the fused attention path: **up to 2.94× training speedup**.
- **Inference** — **diffusion-step caching**: **1.8× speedup**.

Reported inference sweep (Peel Cucumber) over denoise steps with and without caching, tracking success rate, latency, and an SR/latency ratio — an efficiency metric almost nobody else in this cluster reports.

## 6. Does it work?

**Main result:** average action accuracy **+31.7%**; success rate **+31.6%** over Cosmos-Policy, the highest among four strong baselines, with **1.5× speedup**.

**Generalisation** is tested under four OOD shifts (table height +5 cm, OOD spatial placement, unseen mahjong tile suits, altered background), with Dream-Tac ahead of Cosmos-Policy in each — notably including *visual* shifts (background, spatial), where the mechanism must be that contact-grounded prediction stabilises the rollout rather than that touch sees the change.

**Efficiency ablation** separates three configurations — no tactile + no bias, tactile without bias, tactile with bias — showing the bias costs training time that the FlashAttention reformulation recovers.

## 7. What it adds that the others don't

The **non-learned gate**, and the honesty of publishing its trace. Most gating designs here learn a contact probability ([[feelworld]], [[omnivta]]) or a bias from a learned proxy ([[tactile-wam]]); Dream-Tac shows that a frame-difference scalar already tracks the manipulation phase — low on approach, high during contact, low after — and works. It is also the only paper in the cluster to treat the *systems* cost of tactile tokens as a first-class problem, with a FlashAttention-compatible gated bias and an SR-per-latency metric.
