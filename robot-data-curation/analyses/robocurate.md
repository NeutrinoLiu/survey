# RoboCurate — Synthetic-Data Curation & Verification Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | RoboCurate: Harnessing Diversity with Action-Verified Neural Trajectory for Robot Learning |
| **Org** | KAIST (Seungku Kim, Suhyeok Jang, Byungjun Yoon, Dongyoung Kim, John Won, Jinwoo Shin) |
| **Date** | 2026-02 |
| **Artifact** | arXiv 2602.18742 (`paper.pdf`, `paper.html`); seungkukim.github.io/robocurate/ |
| **Disclosure level** | **A — full paper** |
| **Scope** | Curation of **synthetic ("neural trajectory") data**, not web or robot corpora |
| **Stance** | **Verify generated actions by executing them.** Simulation replay as ground truth. |

## 1. The problem it targets

Video generative models can produce diverse task videos from text, and inverse dynamics models (IDMs) can label them with actions — a cheap, unbounded data source. But:

> *"neural trajectory pipelines often face quality issues. Unlike in simulation, the **video generation stage can fail to follow input instructions or produce physically implausible videos** (e.g., objects overlapping or deforming unnaturally), which leads to **incorrect action annotations**. Moreover, **even when videos are accurate, relying on learned models such as IDMs instead of ground-truth annotators can produce low-quality action labels**."*

Two independent failure modes — bad video, and bad labels on good video. Prior work used VLMs to validate the *video*, but *"they have limitations in directly evaluating the actions themselves."*

## 2. The core mechanism — action-level filtering by simulator replay

Each synthetic sample is a pair `(w_gen, a_IDM)`: a generated video and an IDM-predicted action sequence.

```
a_IDM  ──replay in simulator──▶  w_sim(a_IDM)   (rollout video, motion guaranteed consistent with a_IDM)
                                       │
                    compare motion consistency against w_gen
                                       │
                        below threshold ──▶ DISCARDED
```

> *"RoboCurate replays the predicted actions in a simulator and assesses action quality by measuring the **consistency of motion between the simulator rollout and the generated video**."*

The insight: the simulator rollout is a video whose motion **provably** matches `a_IDM`. So comparing it to `w_gen` tests the *label*, not the *pixels*. This closes the gap VLM-based video validators leave open.

**Consistency scorer:** an **attentive probe** over a pre-trained video encoder `f_φ`, trained on positive/negative clip pairs. Negatives are constructed from the real dataset by **inducing temporal shifts** or **sampling video from different episodes** (Fig. 3) — i.e. the discriminator is taught exactly the failure modes of interest (temporal misalignment and wrong-content) using only real data, no synthetic labels required.

**Best-of-N variant:** the same filter can be applied *during* generation rather than after, as rejection sampling — moving compute from generate-then-discard to generate-until-valid.

## 3. Diversity augmentation (the other half)

Filtering alone shrinks a corpus; RoboCurate pairs it with two expansion mechanisms:

| Mechanism | Method | What it varies |
|---|---|---|
| **Image-to-image (I2I) editing** of the initial frame | — | (1) object identity & appearance, (2) lighting, (3) background |
| **Action-preserving video-to-video (V2V) transfer** | Conditioned on **Canny edge videos** to preserve original video structure | Appearance only |

Because V2V *"typically retains the robot motion, we reuse the action annotations labeled by IDMs"* — appearance augmentation for free, no re-labeling. Edge-conditioning is the mechanism that makes "action-preserving" true rather than hoped-for.

**Instruction generation:** a proprietary VLM generates plausible task instructions conditioned on the initial frame. Because *"naive VLM querying may produce wrong instruction templates or physically infeasible robot actions"*, they use **few-shot prompting with examples from the original dataset** to constrain the output distribution. Novel instructions are designed along four axes: **(1) behavior, (2) target object, (3) placement, (4) robot hand type.**

Edited frames are then fed to an image-to-video model to produce the synthetic episode.

## 4. Training use
Imitation learning on `D = D_real ∪ D_syn` — synthetic data supplements rather than replaces real data.

## 5. Results

Relative success-rate improvements over real-data-only:
| Setting | Gain |
|---|---|
| GR-1 Tabletop (300 demos) | **+70.1%** |
| DexMimicGen (pretraining setup) | **+16.1%** |
| **Real-world ALLEX humanoid dexterous manipulation** | **+179.9%** |

The largest gain is in the hardest, most data-starved real-world setting — consistent with the claim that verified synthetic data is most valuable where real data is scarcest.

## 6. What they do not do
- Requires a simulator with a matching asset/robot model — not applicable to in-the-wild human video.
- Verifies *action–video consistency*, not task success; a coherent but wrong behavior can pass.
- Filtering thresholds not published.
- Adds a simulation rollout per candidate sample — meaningful compute cost, partly mitigated by the Best-of-N formulation.

## 7. Transferable takeaways
1. **Execute the label to check the label.** Simulator replay converts an unverifiable annotation into a verifiable one — the strongest form of "action-level" quality control in this survey.
2. **Validate the annotation, not just the observation.** VLM video-quality checks miss IDM labeling errors entirely.
3. **Build discriminators from real-data negatives** (temporal shift, cross-episode sampling) — no synthetic ground truth needed.
4. **Edge-conditioned V2V preserves actions**, so appearance augmentation costs nothing in re-labeling.
5. **Few-shot-constrain generated instructions** with real examples to keep them physically feasible.
6. **Pair every filter with an augmenter.** Filtering shrinks; the combination is what makes synthetic data net-positive.
