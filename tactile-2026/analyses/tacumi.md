# TacUMI — A Multi-Modal Universal Manipulation Interface for Contact-Rich Tasks

**arXiv:2601.14550** · TU Munich + Agile Robots SE + Nanjing University + Shanghai University (T. Cheng, K. Chen, L. Chen, L. Zhang, Y. Zhang, Ling, Hamad, Bing, F. Wu, Sharma, Knoll) · Jan 2026 · [site](https://tac-umi.github.io/TacUMI/)

**One line.** Uses tactile not as a policy input but for **task segmentation** — cutting long-horizon demonstrations into modular skills at boundaries that are invisible to a camera.

## 1. The argument

Long-horizon manipulation is better learned as modular skills plus a coordinator than as one monolithic sequence — but that requires **decomposing** demonstrations, and *"relying solely on visual observations and robot proprioceptive information often fails to reveal the underlying event transitions."*

The motivating example is excellent and specific: *"during cable mounting, operators often **stretch the cable to create tension**. This step is difficult to discern visually but can be clearly captured through tactile signals."* A segmentation boundary with no visual signature at all.

## 2. Hardware

A handheld UMI-derived gripper adding:
- **ViTac sensors** on the fingertips
- **6D force-torque sensor** at the wrist
- **High-precision 6D pose tracker** (Vive)

Plus one mechanical detail that matters: **a continuously lockable jaw mechanism that eliminates the user's grip interference with the F/T reading**. Without it, the wrist force sensor measures the human squeezing the handle rather than the task. That is the kind of design problem that only appears when you actually try to put an F/T sensor on a handheld device.

## 3. Method

A **BiLSTM-based multi-modal segmentation framework** partitioning long-horizon demonstrations into skill sequences by detecting semantically meaningful event boundaries from the temporal structure of the combined modalities.

## 4. Results

On a challenging **cable mounting** task: **>90% segmentation accuracy**, with *"a remarkable improvement with more modalities"* — the modality ablation is the point, showing each added contact channel improves boundary detection.

## 5. What it adds that the others don't

**Tactile for segmentation rather than for control.** This is a distinct role from anything else in the survey — observation ([[contactworld]]), reward ([[tactidex]]), tracking constraint ([[deform360]]), event labelling for offline RL ([[n0-vtla]]), prediction target ([[tacwam]]). Skill boundaries in contact-rich tasks are frequently *contact transitions*, which is precisely what touch measures directly, so the pairing is natural and under-exploited.

The lockable-jaw fix for handheld F/T contamination is also a small, reusable piece of hardware engineering that anyone building a UMI-style contact-sensing device will need.
