# GR00T N1 / N1.7 (NVIDIA Isaac GR00T) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **GR00T N1** (arXiv 2503.14734, 2025-03) → N1.5 → N1.6 → **N1.7** (2026, latest in stream; `groot-n1.7/README.md`) |
| **Org** | NVIDIA |
| **Disclosure level** | **A for N1** (full paper, open checkpoint + training data + benchmarks). **B for N1.7** (model card / release notes only). |
| **Corpus** | Three-layer "data pyramid": web + human video (base) · synthetic (middle) · real robot (top). In-house teleop **88 h → 827 h** via neural augmentation. |
| **Stance** | **Structure the corpus by scale and specificity, then label the unlabeled.** |

## 1. The problem statement — "data islands"

> *"unlike the digital realms of words and pixels, **no Internet of humanoid robot datasets exist** for large-scale pre-training. The data available for any single humanoid hardware would be orders of magnitude too small… However, the great variability in robot embodiments, sensors, actuator degrees of freedom, control modes, and other factors result in **an archipelago of 'data islands' rather than a coherent, Internet-scale dataset**."*

This is the framing the whole cross-embodiment harmonization literature (ABot-M0, ACE-Ego-0, Qwen-RobotManip) is responding to.

## 2. The data pyramid — the organizing abstraction

> *"Rather than treating the training datasets as a homogeneous pool, we organize heterogeneous sources by scale."*

```
        ┌─────────────────────────┐   ← TOP: real-world robot trajectories
        │   real robot teleop     │      smallest quantity, highest embodiment-specificity
        ├─────────────────────────┤      "ensure grounding in embodied, real-robot execution"
        │  synthetic: physics sim │   ← MIDDLE
        │  + neural-generated     │
        ├─────────────────────────┤
        │  web data + human video │   ← BASE: largest quantity, lowest specificity
        └─────────────────────────┘      "provide broad visual and behavioral priors"
```

> *"data quantity decreases, and embodiment-specificity increases, moving from the bottom to the top."*

The pyramid is the most widely adopted mental model in the field, and it is a **curation** claim: the axis along which you organize a heterogeneous corpus should be *specificity*, and each layer has a different job. Compare Cosmos 3's pre-train / mid-train / post-train quality ladder — the same idea applied to quality rather than embodiment specificity.

## 3. Making action-less data trainable — the key mechanism

The base and middle layers have no action labels. Two complementary annotators:

| Technique | Applied to |
|---|---|
| **Latent-action codebook** (learned) | Human egocentric video, neural trajectories |
| **Inverse dynamics model (IDM)** | Action-less videos → pseudo-actions |

> *"These techniques enable us to annotate actions on action-less videos so we can **effectively treat them as additional robot embodiments** for model training."*

Treating an unlabeled video source as *just another embodiment* is elegant: it means one mechanism (embodiment-specific encoders/decoders for variable state and action dimensions) handles both real robots and pseudo-labeled video, with no separate code path.

> *"By unifying all data sources across the data pyramid, we construct a consistent dataset where the input consists of the robot state, visual observations, and language instruction, and the output is the corresponding motor action."*

⚠️ **The unaddressed risk** — and the one ACE-Ego-0 and DYNA-2 explicitly tackle — is that IDM- and codebook-derived labels are *noisier* than sensor logs, yet here they enter the same objective at the same fidelity. GR00T N1 does not report a reliability weighting or a quality gate on pseudo-actions.

## 4. Synthetic augmentation — 10× the peak of the pyramid

> *"we generate synthetic **neural trajectories** using pre-trained video generation models. In this way, we **~10× our in-house collected teleoperation trajectories — the 'peak' of the data pyramid — from 88 hours to 827 hours**, using **diverse counterfactual robot trajectories with novel language instructions**."*

Plus *"diverse simulation trajectories, which also expand the middle part of the pyramid"* — generated with the **Mimic pipeline in Isaac Lab**.

The 88 h → 827 h figure is one of the most concrete augmentation-multiplier disclosures available (compare Qwen-RobotManip's 1,933 h → 24,808 h, a 12.8× via H2R rendering). Note the term **counterfactual**: the synthetic trajectories are not replays but alternative behaviours with new instructions, expanding the language-conditioning distribution as well as the visual one.

⚠️ No verification stage is reported for the neural trajectories — contrast **RoboCurate** (simulator-replay action verification) and **LingBot-VA 2.0** (VLM scoring on semantic preservation + physical plausibility), both of which found generated data needs filtering to be net-positive.

## 5. Mixture sampling
> *"We pre-train our model end-to-end across the three data layers… **by sampling training batches across this heterogeneous data mixture**."*

Embodiment-specific encoders and decoders handle variable state/action dimensions; System 1 (DiT flow-matching) runs at **120 Hz**, cross-attending to System 2 (Eagle-2 VLM) outputs, jointly optimized end-to-end.

## 6. Release posture
GR00T-N1-2B **checkpoint, training data, and simulation benchmarks are publicly released** (GitHub + HuggingFace) — a materially stronger reproducibility position than any of the closed industry entries in this survey, and the reason GR00T serves as the standard baseline in most 2026 comparisons (Qwen-RobotManip, ACE-Ego-0, and others benchmark against GR00T-N1.6/N1.7).

## 7. N1.7 (latest)
Open VLA for generalized humanoid manipulation skills; multimodal language+image input. Released as `nvidia/GR00T-H-N1.7` with model card and repo. ⚠️ Data-side changes from N1 are not documented in a technical report; the corpus description remains the N1 pyramid.

## 8. What they do not do
- No pseudo-action reliability weighting or quality gate (added later by ACE-Ego-0, DYNA-2).
- No verification of neural-generated trajectories (added by RoboCurate, LingBot-VA 2.0).
- No dedup, no per-source rejection rates, no cleaning cascade for the real-robot layer.
- No fitted data-scaling law.

## 9. Transferable takeaways
1. **The data pyramid**: organize a heterogeneous corpus by quantity × embodiment-specificity, and assign each layer a distinct role (priors below, grounding above).
2. **Label the unlabeled and treat it as another embodiment** — latent-action codebooks and IDMs turn action-less video into a first-class corpus with no special-case machinery.
3. **Counterfactual synthetic augmentation** expands the instruction distribution, not just the visual one — 88 h → 827 h.
4. **Release the data and benchmarks with the model.** GR00T's openness is why it functions as the field's reference baseline.
5. ⚠️ **Counter-lesson:** pooling sensor-logged and model-inferred actions at equal loss weight is a known hazard; later works (ACE-Ego-0's reliability weighting, DYNA-2's two-tier routing, π₀.₇'s quality conditioning) all add machinery GR00T N1 lacks.
