# ForceVLA2 — Unleashing Hybrid Force-Position Control with Force Awareness for Contact-Rich Manipulation

**arXiv:2603.15169** · Shanghai AI Laboratory + Tongji + SJTU + Shanghai Innovation Institute + Noematrix + ECNU + Zhongguancun Academy + NUS + Lumos Robotics (Y. Li, Zhaxizhuoma, H. Jiang, Xia, … Lu, Qiao, Pang) · Mar 2026

**One line.** Puts **force in the action space**, not just the observation space — the policy outputs hybrid force–position commands — and reports a clean ablation on *where* in a VLA force should be injected.

## 1. What "tactile" means here

**6-axis force/torque**, at three levels:

- as a **force prompt** into the VLM expert, constructing "force-aware task concepts" across task stages
- as **real-time interaction force** fused in the action expert
- as **part of the output** — the action is a hybrid force–position command

The critique: existing VLAs *"reduce force to an auxiliary perceptual input rather than leveraging it for active, adaptive closed-loop force interaction."* The human analogy is explicit: high-level visual/linguistic reasoning identifies stage-specific targets and coarse spatial relations, while real-time force sensing refines both force and position during interaction.

## 2. Data curriculum — ForceVLA2-Dataset

**1,000 trajectories across 5 contact-rich tasks**: Press bottle, Clean vase, Clean board, Retrieve plate, Assemble gears. Multi-view images, task prompts, proprioceptive state, force signals — plus **force prompts** and **force incorporated into the action space**.

The dataset analysis is genuinely informative about what different contact tasks *are*, in force terms:

| Task | Force distribution | Torque distribution |
|---|---|---|
| Press bottle | concentrated | near zero on all axes — minimal rotational adjustment |
| Clean vase / board | distributed across all three axes (tangential sliding + normal contact) | wide spread — continuous orientation adjustment |
| Retrieve plate | symmetric, centred near zero — gentle exploratory probing | wide spread |
| Assemble gears | broad, especially **z-axis** (alignment and insertion) | **most diverse, especially z** — precise rotational alignment |

They further decompose trajectories into **five short-horizon reactive skills** with explicit numeric definitions:

- **Wipe** — position change > 0.05 m and force amplitude > 10 N
- **Push** — z-position change > 0.05 m and z-force > 5 N
- **Grasp** — z-position change > 0.1 m and force amplitude > 5 N
- **Rotate** — forces on all three axes change > 1 N
- **Explore** — everything else: low-magnitude force with rapid position drift

Skill distribution is dominated by **Explore at 45.62%**, then Wipe, Push, Grasp, Rotate. That number is the force-domain analogue of [[vt-wam]]'s effective-contact-ratio finding: **almost half of a contact-rich dataset is not in meaningful contact.**

## 3. Model

Dual-level, built on π₀:

- **Long-horizon reasoning** — a **force prompt** embedded into the VLM expert builds force-aware task concepts across stages.
- **Short-horizon reactive skill** — a **Cross-Scale Mixture-of-Experts** in the action expert fuses force-aware task knowledge from the VLM with embodied interaction forces, regulating hybrid force–position interaction.

Training: 8× A100, batch 32, 30,000 steps, ~10 hours, AdamW with cosine decay and EMA 0.99. **15 Hz inference on a 4090** with chunk size 30.

## 4. How tactile enters the model — the injection-point ablation

The most transferable result. Three candidate insertion points for the force branch were tested:

| Injection point | Press bottle | Clean vase | Clean board | Retrieve plate | Assemble gears | **Avg** |
|---|---|---|---|---|---|---|
| **VLM pathway** | 10.0 | 10.0 | 5.0 | 0.0 | 0.0 | **5.0** |
| Multimodal encoder (ME) | 75.0 | 55.0 | 65.0 | **40.0** | 55.0 | **58.0** |
| **State fusion** (ME + state) | **80.0** | **75.0** | **70.0** | 35.0 | **70.0** | **66.0** |

**Injecting force into the VLM pathway collapses the model to 5%** — worse than useless, zero on two tasks. Injecting into the multimodal encoder gives 58%, and additionally fusing with the EE 6D pose state gives 66%.

This is independent confirmation, on a completely different signal (6-axis F/T rather than gel images), of what [[at-vla]] found with its Ex1/Ex4/Ex6 rows and what [[n0-vtla]] argues architecturally: **the vision-language prefix is the wrong place for contact information.** Three papers, three mechanisms, same conclusion.

## 5. Does it work?

**+48.0%** over π₀ and **+35.0%** over π₀.₅ across the five tasks, with the authors specifically claiming mitigation of *"arm overload and unstable contact"* failure modes.

**Hardware-agnostic by design.** Execution is grounded in a **Jacobian-based mapping**, decoupling the policy from kinematics and giving compatibility with torque-controlled robots (Franka), robots with EE F/T sensors (UR), and torque-interface actuators (Feetech) — the last enabling low-cost deployment.

**On evaluation methodology**, they state plainly why everything is real-world: *"force interactions are sensitive to friction/contact modeling, which makes simulation results less reliable for our setting. In addition, there is no widely available benchmark that is directly usable for force-aware VLAs."* The second half of that sentence is a fair description of the state of the field — the benchmarks in this survey ([[softvtbench]], [[taco-bench]], [[tacverse]]) target visuo-tactile policies, not force-aware ones.

## 6. What it adds that the others don't

**Force in the action space.** Every other work here consumes force or tactile and emits positions; ForceVLA2 emits a hybrid force–position command, which is what "regulating contact" actually requires. The **injection-point ablation** is the single cleanest piece of evidence in this survey about where contact signals belong inside a VLA, and the **numeric skill decomposition** (with thresholds published) is a reusable way to characterise what is in a contact-rich dataset — including the uncomfortable 45.6% that is just exploration.
