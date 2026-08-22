# Cosmos Policy — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Cosmos Policy: Fine-Tuning Video Models for Visuomotor Control and Planning |
| **Org** | NVIDIA + Stanford |
| **Date** | 2026-01 (arXiv 2601.16163) |
| **Artifact** | `paper.pdf`, `paper.html`; HF `nvidia/Cosmos3-Nano-Policy-DROID` |
| **Disclosure level** | **A — full paper; checkpoints released** |
| **Stance** | **Inherit the data through the backbone.** A single post-training stage on target-platform demonstrations, no architectural changes. |

## 1. The data claim — minimal robot data, maximal inherited prior

> *"Cosmos Policy is a simple approach for adapting a large pretrained video model (**Cosmos-Predict2**) into an effective robot policy through **a single stage of post-training on robot demonstration data collected on the target platform, with no architectural modifications**."*

The curation implication is the key point for this survey: **the expensive, heavily-curated corpus is the video corpus, not the robot corpus.** Cosmos-Predict2/2.5 was trained on 200M curated clips distilled from ~20M hours of raw video by a five-stage pipeline ([Cosmos 3](../cosmos-3/dataprocess.md)); Cosmos Policy then needs only demonstrations from one platform. The cleaning burden has been **paid once, upstream, in a general-purpose corpus**, and amortized across every downstream policy.

This is the same economics as π₀.₇ (~1 h/task), GEN-1 (~1 h/task), Gemini Robotics 2 ("a few hours" to a new embodiment), and mimic's FLUX-mimic (30 min/task) — all of which push the data cost into a pretraining corpus that the robot data never has to match in scale.

## 2. Actions as latent frames — a representation that erases a data-format problem

> *"Cosmos Policy learns to **directly generate robot actions encoded as latent frames within the video model's latent diffusion process**, harnessing the model's pretrained priors and core learning algorithm to capture complex action distributions."*

Encoding actions as latent *frames* means the action data enters through the same channel as the video data. There is no separate action tokenizer, no action-expert head, no per-embodiment encoder — the same simplification LAP achieves with natural language and Hydra-0 achieves with image-plane flow. Three independent 2026 works arriving at "put the actions in a channel the backbone already understands."

## 3. Auxiliary outputs as additional supervision channels

> *"Cosmos Policy also generates **future state images and values (expected cumulative rewards)**, which are similarly **encoded as latent frames**, enabling **test-time planning** of action trajectories with higher likelihood of success."*

Three quantities — actions, future observations, and values — all in one representation. The **value** channel is the RECAP/RoboBrain progress signal, and here it is produced by the same generative process that produces the actions, requiring no separate value-model training data.

## 4. Results
| Benchmark | Score |
|---|---|
| LIBERO | **98.5%** |
| RoboCasa | **67.1%** |
| Real-world bimanual manipulation | **highest average score** |

Beating *"strong diffusion policies trained from scratch, video model-based policies, and state-of-the-art VLA models **fine-tuned on the same robot demonstrations**"* — the last clause matters: the comparison holds robot data fixed, so the margin is attributable to the pretrained video prior, i.e. to the upstream curation.

## 5. Relationship to Hydra-0
[Hydra-0](../hydra-0/dataprocess.md) runs the controlled complement of this experiment: holding the Cosmos 2.5 backbone fixed and swapping its **native relative 6D end-effector action** for image-plane **action flow**, it improves PSNR, SSIM, gripper-flow EPE, FID and FVD on all five datasets. Together the two papers isolate the two variables — Cosmos Policy shows the *backbone prior* matters; Hydra-0 shows the *action representation* matters, independently.

## 6. What they do not do
- No robot-corpus curation pipeline: the demonstration data is target-platform and taken as given.
- No filtering, dedup, or quality gating described.
- No data-scaling curve on the robot side.
- Inherits whatever biases the upstream video curation introduced — and those are optimized for *generation* quality, not for policy quality, a gap noted in the Cosmos 3 entry.

## 7. Transferable takeaways
1. **Pay the curation cost once, upstream.** A heavily curated general video corpus amortizes across every downstream policy; a per-task robot corpus does not.
2. **Encode actions in a channel the backbone already models** (latent frames here, language in LAP, image-plane flow in Hydra-0) and the action-representation pipeline disappears.
3. **Get values for free from the same generative process** — no separate reward-model training data.
4. **Hold robot data fixed when claiming a pretraining benefit**, as this comparison does.
