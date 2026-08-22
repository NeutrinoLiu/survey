# GeoTLM — Geometry-aware Tactile-Language Models for Contact Motion Orientation Reasoning

**arXiv:2606.15909** · Nanyang Technological University (Q. Li, Z. Liu, L. Wang) · Jun 2026

**One line.** Finds that state-of-the-art tactile encoders are **at chance (~50%) telling clockwise from counterclockwise rotation** on GelSight Mini, and fixes it with a **14K-parameter** geometric module — a failure of inductive bias, not capacity.

## 1. The finding

The preliminary experiment is the paper's real contribution. Classifying a single GelSight Mini contact as clockwise or counterclockwise rotation, *"large tactile and vision backbones probed under a frozen protocol stay close to chance (~50%), even though the same backbones perform well on in-distribution recognition."* This includes **Sparsh and AnyTouch 2** — the field's leading tactile encoders.

The diagnosis is precise: *"The signal needed to separate the two rotation directions is present in the contact, but the pretrained representation does not expose it in a form a classifier can read."*

And the interpretation: *"a failure of inductive bias rather than capacity: an 87M-parameter tactile encoder and larger vision backbones alike miss a geometric signal that a compact, structured model can exploit."* Generic V-JEPA and contrastive objectives *"are not designed to preserve such antisymmetric shear at the patch-grid resolution used by downstream tokens. Therefore, adding more frames or more parameters does not necessarily recover physical information that has already been compressed away."*

That is a direct challenge to the scaling premise of the entire representation-learning cluster ([[anytouch2]], [[htt]], [[ftp-1]], [[tactx]]), and it complements [[anytouch2]]'s own finding that alignment objectives suppress dynamic perception.

## 2. Model — Differentiable Geometric Representation

**14K trainable parameters**, operating on **frozen** tactile patch tokens:

1. **Contact-mask-guided shear representation** learned in the shear field
2. **Antisymmetric seven-region pooling**, motivated by the physical intuition that *"rotational contact produces opposite-sign deformation patterns across spatial regions"*

The design rationale is explicitly a middle path. Not generic tokens (which discard the geometry), and not handcrafted physics either: *"directly imposing handcrafted physical operators such as curl, divergence, or strain is not enough either, as such operators can be task-specific and brittle across diverse contact patterns and sensor resolutions."*

Instead: *"preserve and structure tactile shear-field geometry before language-level reasoning, rather than forcing low-resolution tactile tokens into fragile closed-form physics operators."* The geometry is preserved in a form the language head can **flexibly interpret and recombine**.

## 3. Results

| Task | Improvement over same backbone w/o geometric encoder |
|---|---|
| **Novel-object rotation** accuracy | **+14.6%** |
| **Real-sensor sliding** accuracy | **+16.2%** |

Both on tasks where the unaugmented backbones sit near chance.

## 4. Why it matters

The motivating tasks are not artificial: *"Tightening a bottle cap requires identifying rotation direction at each step, peg insertion demands early detection of incipient slip, and safe grasping relies on estimating contact force before the object is crushed."* These are core manipulation primitives, and the finding is that the field's best tactile representations cannot support the first of them.

## 5. What it adds that the others don't

A **falsifying probe** on the dominant tactile-pretraining recipe, plus a minimal, physically motivated fix. The 14K-parameter module recovering what an 87M-parameter encoder discarded is the strongest argument in this survey that **tactile representation learning needs geometric inductive bias, not more data** — and the antisymmetric-pooling design is a concrete example of what that bias looks like. Read against [[tacgen]], which shows tactile is necessary evidence, and [[anytouch2]], which shows the standard objective trades away dynamics: three independent results converging on the same conclusion about what current tactile encoders lose.
