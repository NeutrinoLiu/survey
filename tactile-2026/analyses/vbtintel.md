# Vision-Based Tactile Intelligence for Robotics — Sensing, Learning, and Embodied Manipulation

**arXiv:2608.15490** · Great Bay University + Tsinghua + HKU + NTU + PolyU + SCUT + KTH + KCL (P. Zhou, J. Hu, S. Chen, Z. Zhang, H. Ma, Z. Lu, S. Liu, X. Wang, P. Zheng, X. Li, **S. Luo**, **J. Pan**, Navarro-Alarcon, **C. Yang**, **M. Y. Wang**) · Aug 2026

**Framing:** treats *"sensing hardware, learning methods, simulation, and datasets as an **integrated sensing-and-learning system**"* rather than as separate literatures.

## The case for VBTS

Traditional electronic tactile sensors — piezoresistive, capacitive, magnetic, MEMS biomimetic, piezoelectric, triboelectric, EIT — *"often remain limited in spatial resolution, signal richness, and primarily measur[e] normal or triaxial force rather than rich geometric and textural contact information."*

VBTS *"transform contact-induced deformation into tactile images"* and are therefore *"naturally compatible with modern image-based learning frameworks"*, connecting contact physics to computer vision, multimodal learning, and *"emerging tactile vision-language-action models."*

The historical lineage is well traced: image-based membrane reconstruction → optical tracking of coloured markers for distributed force → transparent elastic fingertips → **GelSight** (painted elastomer + structured illumination) → **TacTip** biomimetic family → **DIGIT** (Meta AI × GelSight).

## Three-part structure

1. **Hardware taxonomy** organised by **deformable elastomer design, sensor size and shape, and optical system design** — explicitly *"to guide future sensor development"*, i.e. as a design space rather than a catalogue.
2. **Hierarchical view of tactile intelligence**, from low-level signal understanding → task-level policies → **foundation models**.
3. **Simulation platforms and datasets as a scaling layer**, together with **sim-to-real transfer and cross-sensor adaptation**.

That third grouping is the survey's best organising choice: it treats simulation and datasets as serving the same function — supplying data the hardware cannot — which is exactly the relationship the 2026 simulation cluster ([[univtac]], [[tactilegenesis]], [[etac]]) and generation cluster ([[felt]], [[unitac]], [[vq-touch]]) bear to each other.

## Position in the survey

The most **hardware-grounded** of the three 2026 surveys, and the right entry point for understanding why sensors differ and what that costs downstream — the problem [[tacverse]] measures, [[htt]] and [[tactx]] engineer around, and [[uniforce]] dissolves by changing the alignment target. Its August 2026 cutoff makes it the most current of the three.
