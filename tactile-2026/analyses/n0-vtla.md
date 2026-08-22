# N₀-VTLA — Scaling Vision–Tactile–Language–Action Model with Latent Tactile Tokens

**arXiv:2607.23782** · NeoteAI Team & Fudan TEAI Team · Jul 2026 · [code](https://github.com/neoteai/N0-VTLA) · [site](https://research.neoteai.com/n0-vtla/)

**One line.** Argues that both existing ways of adding touch to a VLA are wrong for the same reason, and takes a third path: **keep tactile out of the vision-language prefix entirely, and condition the action expert on a *prediction* of contact change rather than the current reading.**

## 1. What "tactile" means here

Vision-based tactile on each gripper finger, but never used raw. The policy sees **contact-difference images**: for view k, the episode-start baseline frame `tac^k_0` (the zero-contact reference) is subtracted from the current frame `tac^k_τ` **in pixel space**, and the difference is encoded.

The characterisation of the signal is the sharpest in the survey: *"This signal is nothing like a camera view: tactile frames are noisy, nearly empty away from contact, and informative almost only in the brief windows when contact forms or is about to change."*

Differencing against a per-episode baseline **removes static gel appearance and much of the mount-specific imprint**, making the representation robust to (though not invariant under) sensor-placement differences — a cheap, effective alternative to the learned cross-sensor token spaces of [[ftp-1]] and [[htt]].

## 2. The critique of prior integration paths

Two existing paths, both rejected:

1. **Concatenate tactile tokens into the vision-language context** (OmniVTLA, Tactile-VLA) — *"a signal that is sparse and mostly silent buys little in a prefix built for information-dense views."*
2. **Inject the current tactile reading into the action pathway** to guide denoising — *"this placement mischaracterizes the role of touch, since a tactile frame records contact that actions already taken have produced... Conditioned on it alone, the policy stays one step behind its own contact events."*

That second sentence is the cleanest statement of the reactive-tactile problem anywhere in this literature.

## 3. Data curriculum

**NeoData** — a large-scale multi-platform visuo-tactile corpus spanning single- and dual-arm configurations, made trainable as one model by a **canonical cross-embodiment action space** (a fixed 32-dimensional container; each platform populates the dimensions its embodiment uses, the rest carry zero targets the model learns to reproduce, so single- and dual-arm data coexist under one fixed-width flow-matching objective).

**Three-stage recipe** for bringing the newly initialised tactile pathway online without destabilising the pretrained π₀.₅ backbone:

- **Stage 1** — train the predictor alone against a future-tactile target; loss backpropagates **into the predictor only**.
- **Stage 2** — freeze the tactile perception stack and **mask the vision-language pathway**, so the action expert learns to consume the latents in isolation.
- **Stage 3** — train the full policy end to end.

Then **ALTER** for offline policy improvement (below).

## 4. How tactile enters the model — latent tactile tokens

**Encoder.** `g_k = f_enc(tac^k_τ − tac^k_0) ∈ R^{10×d}` — a **frozen** self-supervised visual encoder (DINOv2) plus a **trainable linear projection** to the backbone's token width. Each tactile image yields **10 tokens**: one class token and nine spatial tokens from a **3×3 adaptive average pool** over the encoder's 16×16 patch grid. Tokens from n active views concatenate to `g ∈ R^{10n×d}`.

Freezing is deliberate and triple-justified: it preserves the self-supervised representation, **lets an unseen sensor be onboarded by training only the lightweight projection**, and removes encoder activations and optimizer state from the training memory budget.

**Predictor.** A lightweight module reads `g` in the context of the contextualised vision-language prefix and distils it into learned latent queries → **latent tactile tokens z**. When an episode has no tactile at all, a **learned null token** stands in, so z is always produced and the policy degrades gracefully to vision-language control.

**The target is what makes z predictive rather than descriptive:**
```
z* = (1/n) Σ_k f_enc( tac^k_{τ+H} − tac^k_τ ) ∈ R^{10×d},    H = 50
```
i.e. the same encoder applied to the tactile **change over the next H steps**, averaged across active views. So z estimates *the net contact change the coming action chunk will cause*.

**Placement.** z conditions the **flow-matching action expert directly**; tactile **never enters the vision-language prefix**. Action horizon H = 50.

## 5. ALTER — advantage-conditioned offline RL

For learning from stored deployment data. A **pairwise progress model** is trained from three supervision sources:
- clean demonstrations (dense progress pairs)
- **tactile-detected object-drop events**
- logged human-in-the-loop corrections

These produce **stage-relative binary advantage labels** for offline policy learning. Stages are weighted by **demonstrated duration** rather than spaced uniformly by elapsed time (the contrast the authors draw with χ₀), and local comparison supervision is added around the drop events and corrections.

Note that tactile does double duty here: as the policy's predictive conditioning **and** as the event detector that labels the offline RL data.

## 6. Does it work?

**Representation-level check first** — a discipline more papers should adopt. After **Stage 1 alone**, the latent tokens retrieve their matching future-tactile target at **92.3% top-1 accuracy against a 3.2% chance level**. The latents are genuinely grounded in future contact before any closed-loop claim is made.

**Closed loop:**

| Benchmark | N₀-VTLA | Strongest baseline |
|---|---|---|
| 20-task simulation suite | **63.8%** | 44.0% |
| NeoReal real-robot (9 tasks) | **wins all 9** | — |
| 3 long-horizon real tasks + ALTER | **75–95%** | — |

A **+19.8 point** margin on the simulation suite, and unbeaten across nine real tasks. ALTER applies to both the base VLA and N₀-VTLA, with N₀-VTLA+ALTER highest on all three long-horizon tasks.

**Stated future work.** The authors note that the free-latent and supervised variants are *"two points in a broader design space"* and explicitly generalise the claim beyond their architecture: **conditioning action generation on a compact estimate of net contact change over the chunk horizon, rather than on raw sensory tokens**, is the transferable idea.

## 7. What it adds that the others don't

Three separable contributions. **(a)** The placement argument — tactile belongs in the action pathway as a prediction, not in the language prefix as an observation — supported by an explicit representation-level probe. **(b)** The **contact-difference-plus-frozen-DINOv2** encoder, which handles cross-sensor onboarding by training one linear projection, a far cheaper answer than the token-space unification of [[ftp-1]] or [[htt]]. **(c)** Using tactile as an **automatic event labeller** for offline RL, which is a use of touch nothing else in this survey exploits. Read alongside its sibling [[n0-twam]], which predicts touch *jointly with vision* in a world model; N₀-VTLA discards the visual future entirely and keeps only the tactile one.
