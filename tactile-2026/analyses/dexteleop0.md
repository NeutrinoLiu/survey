# DexTeleop-0 — Force-Aware Bimanual Dexterous Teleoperation with Ego-Centric Perception

**arXiv:2606.23431** · Nanyang Technological University + OOJU (H. Liu, Y. Jiang, H. Park, Y. Xue, Z. Wang) · Jun 2026 · [site](https://henryhcliu.github.io/dexteleop-0)

**One line.** Puts **tactile force balancing inside the teleoperation tracking loop** — the operator's coarse intent is projected onto a physically compliant manifold in real time, across **56 concurrent DoF**.

## 1. The problem

*"Traditional teleoperation systems often fail in contact-rich tasks because embodiment gaps hinder accurate kinematic mapping, while tactile and force feedback remain absent."* The operator cannot perceive subtle contact, and the mapping cannot be made exact.

The core principle is illustrated cleanly: **before balancing**, misaligned finger contact points generate asymmetric tactile forces that fail to counteract friction and gravity → unstable interaction. **After balancing**, the hand's posture is dynamically adjusted from real-time tactile feedback to rectify contact forces → stable, force-balanced interaction.

## 2. Method — tactile-driven adaptation

A **real-time optimisation loop** translating coarse human tracking intent into precise, force-compliant robot commands:

1. Estimate accurate **contact points** from tactile
2. Use a **tactile-enabled fingertip force-sensing profile**
3. Compute **localised corrections** via the **operational space Jacobian with respect to joint angle updates**

Coordinating **56 concurrent DoF** across dual arm-hand assemblies.

## 3. Results

Evaluated in high-fidelity simulation and on physical hardware, against representative baselines, on **robust grasping, disturbance-resilient manipulation, and complex dexterous tasks**: consistently higher success and better execution efficiency, with *"outstanding grasp stability, anti-disturbance compliance"* and *"drastically reduce[d] destructive physical interaction forces"* relative to position-bound or decentralised joint-level baselines — while maintaining kinematic fidelity.

## 4. The stated insight

> *"incorporating localized tactile corrections and physical force-balancing directly into a tracking optimization loop is **more critical for closing the embodiment gap and ensuring interaction safety than merely increasing baseline tracking resolution**."*

That is a useful claim, and it cuts against the direction most teleoperation work takes. The instinct is to track the human more precisely; DexTeleop-0 argues that precision in the wrong space (joint angles) cannot substitute for correctness in the right one (contact forces), because the two hands are not the same shape.

Future work: predictive slip detection, and using the collected visual-tactile trajectories to train imitation policies for industrial and laboratory assembly.

## 5. What it adds that the others don't

**Tactile as a constraint on the teleoperation mapping itself**, rather than as feedback to the operator ([[vihateleop]], [[haptile]], [[omniumi]]) or as recorded data ([[dexvitac]], [[art-glove]]). By closing the loop inside the retargeting optimisation, it makes the *system* responsible for physical validity rather than the human — which is the only approach that scales to 56 DoF, where no operator could balance forces manually. The force-reduction result also makes it a safety contribution: it is the teleoperation analogue of [[vitar]]'s bounded-residual argument.
