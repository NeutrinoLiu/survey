# FingerEye — Learning Dexterous Manipulation with Continuous Vision-Tactile Sensing

**arXiv:2604.20689** (v3, Jun 2026) · National University of Singapore + RoboScience + HUST + SCUT (Z. Xu, Y. Li, X. Wu, Qiu, **L. Shao**) · Apr 2026 · [site](https://nus-lins-lab.github.io/FingerEyeWeb/)

**One line.** Identifies the **transition from seeing to touching** as the weakly observed moment in dexterous manipulation, and builds a fingertip sensor that works on both sides of it — binocular cameras before contact, marker-tracked deformation after.

## 1. The gap

*"Manipulation refers to an agent's control of its environment through **selective contact**."* Selective contact requires sensing that is informative **before contact, during contact initiation, and after contact**.

The worked example: standing a coin upright on a table requires localising *"the thin coin edge before contact, detect[ing] fingertip contact onset during contact initiation, and regulat[ing] contact forces after contact to prevent slipping or toppling."*

But existing systems use two separate sources: external vision (which *"provides indirect observation to contact"*) and tactile (which works *"only after contact is established"*). *"This separation leaves the most fragile moment in dexterous manipulation — the transition from seeing to touching — weakly observed."*

## 2. Sensing

**Binocular RGB cameras + a compliant contact interface** in one fingertip:
- **Before contact** — fingertip cameras give close-range visual cues and **implicit stereo** for precise approach and object localisation
- **After contact** — **marker-tracked deformation of the compliant ring** acts as a proxy for **contact wrench** sensing

One device, continuous coverage across the transition. In the See-Through-Skin lineage, but with binocular stereo rather than a switchable membrane.

## 3. Learning — and a real fusion problem

**FingerEye Policy** applies **group-structured modality fusion** to *"reduce **modality shortcuts** and better exploit distributed fingertip feedback."*

The shortcut problem is specific to multi-fingertip sensing: with several fingers each providing vision *and* tactile, a policy can latch onto whichever channel is easiest and ignore the rest. Group structure prevents that — the same modality-collapse concern [[restacvla]] and [[vtam]] address across modality *types*, here arising across sensor *instances*.

Supporting infrastructure: **real-and-sim** data collection and evaluation, with a systematic study of policy-interface designs for multiple FingerEye sensors.

## 4. Results

Across **seven contact-sensitive task settings**, FingerEye improves over a wrist-only policy by **over 30 percentage points** in mean success, **in both simulation and the real world**.

Open-source hardware and software released.

## 5. What it adds that the others don't

**Continuity across the contact boundary.** Every other sensor in this survey is either a camera that stops being useful at contact or a tactile sensor that starts being useful at contact; FingerEye spans both from the same optical hardware, which is precisely where the failures concentrate. It is also the only entry to identify **modality shortcuts across distributed fingertips** as a learning problem, and to design the fusion structure against it.
