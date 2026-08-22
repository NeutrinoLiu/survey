# FeelWorld — Visuo-Tactile World Model for Hierarchical Contact Prediction and Planning

**arXiv:2607.24267** · Institute of Automation CAS + Imprintx Robotics + BAAI (Ma, C. Zhang, Xue, Cai, Yao, Cui, S. Wang) · Jul 2026

**One line.** Argues that "tactile" is not one prediction target but **three at different levels of abstraction** — contact, deformation, slip — and that fusing touch during free-space motion is fusing sensor noise.

## 1. What "tactile" means here

**3D tactile point clouds** `P_t`, encoded by a frozen **FG-CLTP** encoder (the authors' own contrastive language-tactile pretraining model) into `h_t = E_τ(P_t) ∈ R^{d_τ}`, capturing contact area, pressure distribution, and principal deformation direction.

One implementation detail with a real justification: **LayerNorm is deliberately omitted for `h_t`**, because normalisation would remove the magnitude information that encodes contact intensity. Most fusion designs normalise reflexively; this is a case where doing so destroys the physics.

The paper's decomposition — the central idea:

| Level | State | Nature | Role |
|---|---|---|---|
| 1 | **Contact** `c_t ∈ [0,1]` | binary, immediate | **gates** visuo-tactile fusion |
| 2 | **3D tactile latent** `h_t` | continuous, spatial | represents local interaction geometry |
| 3 | **Slip** `s_t ∈ [0,1]` | binary, **inherently temporal** | enters the planning cost |

Prediction proceeds in that order, and each level is **explicitly supervised** rather than left implicit in a reconstruction loss.

The framing of modality asymmetry is the sharpest in the survey: *vision supplies continuous scene information, whereas tactile signals carry physical meaning **only during contact***. Everything else follows from taking that literally.

## 2. Data curriculum

Three contact-rich tasks: **chip grasping, fruit grasping, USB insertion**. Multi-view visual (third-person for global context, wrist for interaction feedback), 3D tactile point clouds, robot proprioception `q_t`, and 7-dimensional actions.

Training: joint visual + tactile latent prediction, explicit contact and slip supervision (focal loss), **autoregressive rollout with context noise injection** to build robustness against compounding error.

## 3. Model

Frozen encoders on both sides — **V-JEPA** for vision (`o_t = E_v(I_t)`), FG-CLTP for touch — feeding a trainable dynamics transformer:

```
ẑ_{t+1} = [ẑ^o_{t+1} ; ẑ^τ_{t+1}] = W_θ(z_t, g_t, q_t, a_t)
```

`ẑ^o` decodes to the next visual latent; `ẑ^τ` decodes through three heads to `(ĉ_{t+1}, ĥ_{t+1}, ŝ_{t+1})`. `g_t` is the contact gate.

## 4. How tactile enters the model — contact-gated asymmetric attention

Three distinct attention pathways, deliberately not symmetric:

1. **Visual tokens keep a visual-only self-attention pathway** — an appearance-driven prediction path that survives regardless of contact state.
2. **Tactile tokens attend to visual tokens** — grounding local contact geometry in global scene context. This direction is *always on*.
3. **Visual tokens query tactile features through cross-attention, gated by predicted contact probability `g_t`** — suppressed in free space, enabled during contact.

So touch can always look at vision, but vision can only look at touch when there is something to look at. This is precisely the design [[n0-twam]] argues against ("gating tactile tokens to protect the visual stream") and precisely what [[tactile-wam]] independently converges on. The disagreement is live and both sides have numbers.

## 5. Experiment setup

**Contact-aware CEM planning.** The cost function is itself hierarchical:

```
J(A) = Σ_{k=1..H_p} [ D_v(ô_{t+k}, o_goal) + g_k ( w_τ D_τ(ĥ_{t+k}, h_goal) + w_s D_slip(ŝ_{t+k}) ) ]
```

The predicted contact gate `g_k` multiplies the entire tactile cost term, so planning naturally divides into a **vision-guided approach stage** and a **joint visuo-tactile optimisation stage**. Slip enters as a penalty. CEM with N candidates, top-K elites, I iterations.

Baselines: vision-only world model, and a **naive (symmetric) visuo-tactile** variant — an important control, since it separates "adding touch" from "adding touch this way".

## 6. Does it work?

**Prediction quality.** 10-step LPIPS drops **0.084 → 0.058**. After an **80-step** autoregressive rollout, LPIPS stays **61% lower** than the visual baseline — the long-horizon number matters more, since compounding error is where vision-only world models fail. Contact F1 **98.1%**, slip F1 **83.4%**.

The slip/contact asymmetry is worth noting: contact is nearly solved (98.1%) while slip sits at 83.4%. Slip is the temporal judgement, and it remains the hard one — consistent with [[deform360]]'s finding that normal-axis tactile cannot see micro-slip at all.

**Gate ablation.** Removing the contact gate degrades **SSIM 0.884 → 0.876** and **PSNR 28.89 → 27.52**. Modest in absolute terms, but note what it means: an *unfiltered* tactile stream actively **degrades visual prediction**, because free-space tactile readings are noise being treated as evidence.

**Planning** (zero-shot, three tasks): average success **81.7%**, **+32.5 points** over the vision-only baseline, and above the naive symmetric visuo-tactile variant across all three tasks.

## 7. What it adds that the others don't

The **hierarchy with explicit supervision at each level**. Most tactile world models predict one undifferentiated tactile target and hope the latent organises itself; FeelWorld's argument — that dense visual signals otherwise overshadow sparse but critical tactile information, and that unstructured touch is weak supervision for latent dynamics — is supported by the gate ablation and the F1 gap between contact and slip. The gated CEM cost, where the predicted contact probability multiplies the tactile cost term, is also the neatest way in this survey to make a planner switch regimes without a hand-written state machine. Compare [[contactworld]], which studies *which representation* to use; FeelWorld studies *when to use it*.
