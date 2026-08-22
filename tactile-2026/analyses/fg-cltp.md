# FG-CLTP — Fine-Grained Contrastive Language Tactile Pretraining for Robotic Manipulation

**arXiv:2603.10871** · Institute of Automation CAS + BAAI (Ma, C. Zhang, Cai, Yao, Cui, S. Wang) · Mar 2026

**One line.** Names the **lexical bottleneck** in tactile-language models — words can say "a hard press" but not "5 N versus 20 N" — and fixes it by tokenising the numbers themselves.

## 1. The problem

Existing tactile-language alignments are **qualitative**: they *"predominantly rely on qualitative descriptors (e.g., texture), neglecting quantitative contact states such as force magnitude, contact geometry, and principal axis orientation, which are indispensable for fine-grained manipulation."*

The **lexical bottleneck**, stated precisely: *"While standard linguistic tokens can effectively describe categorical attributes (e.g., recognizing a hard press), they fail to capture the continuous, high-precision physical parameters essential for control, such as distinguishing whether a contact force is 5 N or 20 N, or quantifying the precise penetration depth in millimeters. This lack of quantitative-semantic alignment severs the link between high-level reasoning and low-level execution."*

They also argue for **3D over 2D tactile**: *"2D tactile images are typically sensor-specific, as they entangle contact geometry with internal illumination patterns, which complicates cross-sensor generalization. In contrast, 3D point clouds explicitly capture spatial deformation while omitting hardware-specific artifacts, providing a sensor-agnostic representation."* That is the same disentangling argument [[unitac]] makes (sensor configuration vs object properties) and [[uniforce]] makes (appearance vs force), reached through geometry.

## 2. Data and method

**>100k tactile 3D point-cloud–language pairs** explicitly capturing multi-dimensional contact states *from the sensor's perspective*.

**Discretised numerical tokenisation** injects explicit physical metrics into the multimodal feature space. Descriptions read like:

> *"cylindrical, pressed, at `<pos_12_12>`, `<depth_2.1>`, oriented `<ori_240>`"*

— so position, penetration depth and principal-axis orientation become tokens the language model can align against, rather than adjectives.

Downstream: **3D-TLA**, a 3D tactile-language-action architecture with a **flow-matching** policy, using the frozen FG-CLTP encoder.

## 3. Results

| Metric | Result |
|---|---|
| Classification accuracy | **95.9%** |
| Regression error (MAE) | **−52.6%** vs SOTA |
| **Sim-to-real gap** | **3.5%** |

The 52.6% MAE reduction is the number that matters — it is the *quantitative* axis the lexical bottleneck was blocking. And the **3.5% sim-to-real gap** is a direct consequence of the 3D point-cloud choice: geometry transfers where gel appearance does not.

Downstream 3D-TLA significantly outperforms strong baselines on contact-rich tasks (tube insertion, blackboard cleaning), with demonstrated cross-sensor generalisation.

## 4. What it adds that the others don't

**Numbers as tokens.** Every other tactile-language model in this survey aligns touch to natural-language *adjectives*; FG-CLTP observes that adjectives cannot carry the precision control needs, and discretises physical quantities into the vocabulary. It is the tactile instance of a general problem in embodied language models — semantic descriptions do not close the loop to continuous action — and the 52.6% regression improvement is direct evidence.

Its 3D point-cloud representation also gives it the smallest published sim-to-real gap in this survey (3.5%), which makes it the natural encoder for [[feelworld]] and [[hitac-wam]] — both of which use FG-CLTP precisely for that reason.
