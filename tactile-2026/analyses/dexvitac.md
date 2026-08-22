# DexViTac — Collecting Human Visuo-Tactile-Kinematic Demonstrations for Contact-Rich Dexterous Manipulation

**arXiv:2603.17851** · Huazhong University of Science and Technology + Wuhan Huaweike (X. Chen, Pan, M. Li, X. Ding) · Mar 2026 · [site](https://xitong-c.github.io/DexViTac/)

**One line.** A wearable rig capturing **first-person vision + five-finger tactile + hand kinematics in the wild** at **248 demonstrations/hour**, paired with a representation idea: local tactile is meaningless without the global hand pose that produced it.

## 1. The gap in collection hardware

Each existing option fails differently: teleoperation and AR/VR are *"restricted to structured environments and lack multi-finger tactile feedback"*; UMI-style embodiment-agnostic frameworks are *"primarily limited to low-degree-of-freedom grippers"*; exoskeletons are *"intrusive to natural motion or lack essential contact information."*

DexViTac integrates a **177° FOV fisheye camera**, **motion-capture gloves**, **high-resolution tactile sensors on all five fingers** (thumb, index, middle, ring, pinky) and a **T265** tracker — *"without impeding natural human motion."*

## 2. The representation problem — and the fix

The observation is sharp: *"High-fidelity tactile signals exhibit inherent **spatial semantic ambiguity**, where local signals lack physical meaning when dissociated from the global kinematic configuration of the hand."*

A pressure blob on a fingertip means nothing without knowing where that finger is and what the other four are doing. So they propose **kinematics-grounded tactile representation learning**: a self-supervised framework *"grounding local tactile features within a global kinematic latent space."* Then a two-stage training strategy on top.

This is a genuinely different unifying signal from the rest of the cross-sensor cluster — not physical units ([[uniforce]]), not functional area ([[ftp-1]]), not contrastive pairing ([[tactx]]), but **hand configuration as the context that disambiguates touch**.

## 3. Dataset and results

**2,400+** visuo-tactile-kinematic demonstrations, at **>248 demonstrations/hour** — the highest collection throughput reported in this survey, and the number that justifies the wearable form factor.

Robust against complex visual occlusion. Real-world deployment: **>85% average success across four challenging contact-rich tasks**, significantly outperforming baselines. Tasks include **liquid transfer** and **in-hand action**.

Hardware designs, dataset and codebase to be open-sourced.

## 4. What it adds that the others don't

**Five-finger tactile in the wild at scale**, and the kinematics-grounding argument. Most human-side tactile collection here is glove pressure without full hand kinematics ([[egotac]], [[touchanything]]) or robot-side teleoperation ([[haptile]], [[prism-ind]]); DexViTac captures the full triple and then argues — correctly — that the tactile half is uninterpretable without the kinematic half. The 248 demos/hour figure also sets a useful bar: at that rate, the 160-hour scale of [[hcttp]] is roughly two weeks of one person's work.
