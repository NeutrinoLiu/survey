# Learning Physical Interaction — A Survey of Tactile- and Force-aware Robot Learning

**arXiv:2608.07558** · NTU + Stanford + Berkeley + MIT + NUS + Georgia Tech + Tokyo + ETH + Harvard + Imperial + KTH + TU Darmstadt (Shan, C. Zhou, R. Wang, … Soh, Na Li, Johns, **Kragic**, **Peters**, Matusik, Tomizuka, **Malik**, J. Yang) · Aug 2026 · [collection](https://github.com/NTUMARS/Awesome-Tactile-Force-aware-Robot-Learning) · [site](https://lorenzo-0-0.github.io/tactile-force-survey/)

**The gap it fills:** *"existing surveys have not explicitly reviewed force- and tactile-aware robot learning from a unified perspective that jointly captures **multimodal sensing and multi-phase system design**."*

## The framing

*"When robots move from free-space motion to physical interaction, **the most task-critical information often shifts from visual observations to interaction feedback**. Under contact, the target object may be occluded, its state may change due to deformation, and successful execution may depend on where contact occurs, whether it is stable, and how large the interaction force becomes."*

## TF-ART — the taxonomy

A **Tactile/Force-Aware Robot learning Taxonomy** structured as **phases**, which is the right axis given how much of the 2026 literature is about *where* in the pipeline touch enters:

| Phase | Content |
|---|---|
| **Phase 0** | Multi-modality perception — vision, force/torque, tactile, language, state |
| **Phase 1** | Multi-modality **fusion** into intermediate modalities |
| **Phase 2** | Policy for **primary actions** (action chunks) |
| **Phase 3** | Policy for **refined actions** → **reactive robot control** with control stiffness K and reference force F_ref, closed-loop force control |

Plus branching modules for **input reconstruction, missing-modality prediction, and explicit force-control algorithms** — the last covering privileged-state/dynamics modelling with regressor/student networks, i.e. the distillation line ([[ptld]], [[tacblr]]).

This phase decomposition maps directly onto the disagreements in this survey: [[forcevla2]] argues about Phase 1 placement, [[ratg]] about where in Phase 2 supervision lands, [[retouch]] and [[tubedp]] about Phase 3 refinement, [[cgp]] about the Phase 3 → controller interface.

## The forward-looking argument

Two claims worth extracting.

**Compliance can be hardware, not software.** *"An orthogonal route is to **embed compliance directly in the hardware**, through soft or tendon-driven actuation, antagonistic variable-stiffness joints, and inherently compliant anthropomorphic hands. Such embodiments absorb impact, distribute contact, and stabilize interaction passively, **reducing the bandwidth and accuracy demanded of force sensing and feedback control**, and in some cases enabling safe contact-rich behavior with little or no explicit force sensing."*

That points at **body–policy co-design**: *"morphology, actuation compliance, and learned control are optimized together rather than layering a policy onto a fixed platform."* And it reframes the sensing problem — *"intrinsic compliance changes how contact forces are transmitted to and observed by the available sensors."*

**Transferable physical concepts.** *"Tactile- and force-grounded robot intelligence should move beyond learning task-specific motion patterns and toward acquiring **transferable physical concepts, such as contact, pressure, friction, and stability**."*

The stated requirements: unified multimodal representations, phase-aware policy architectures, systematic policy-control integration, compliant embodiment and body-policy co-design, stronger awareness of world evolution, and **standardized benchmarks for real-world physical interaction**.

## Position in the survey

The most **architecturally** organised of the three surveys, and the one whose taxonomy most directly matches how the 2026 methods actually differ. Its author list — Kragic, Peters, Malik, Tomizuka, Matusik, Soh, Johns — also makes it something close to a field-level statement of where tactile robot learning believes it is.
