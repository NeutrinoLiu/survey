# UniVTAC — A Unified Simulation Platform for Visuo-Tactile Manipulation Data Generation, Learning, and Benchmarking

**arXiv:2602.10093** · ScaleLab SJTU + D-Robotics + ViTai Robotics + HKU + Nanjing + Shenzhen + Wuhan + Fudan + Tsinghua (B. Chen, W. Wan, T. Chen, X. Guo, … W. Sui, Y. Mu) · Feb 2026 · [site](https://univtac.github.io/)

**One line.** Three things in one release — a **simulator**, an **encoder pretrained on its synthetic data**, and an **eight-task benchmark** — which is why it becomes the default evaluation platform for much of the 2026 world-model and VLA work.

## 1. The gap it targets

Two coupled bottlenecks. **Data**: real tactile hardware suffers *"the lack of standardized sensor designs, high manufacturing and deployment costs, and the difficulty of large-scale production and data collection."* **Evaluation**: *"the absence of unified and comprehensive benchmarks for visuo-tactile manipulation hinders systematic evaluation and iterative improvement."*

## 2. The three components

**UniVTAC simulator** — supports **three widely used visuo-tactile sensors**, generating pressure patterns, marker deformations, and tangential force cues at scale with controllable interactions.

**UniVTAC Encoder** — trained on the synthetic data under **multi-task supervision**: visuo-tactile image reconstruction *and* pose estimation. The design intent: representations that *"capture fine-grained contact boundaries while remaining sensitive to object pose and interaction dynamics."*

**UniVTAC Benchmark** — eight representative visuo-tactile manipulation tasks (insert tube, insert USB, lift bottle, bottle upright, put bottle, insert HDMI, grasp classify, pull-out key).

Policy architecture: transformer, **4 encoder layers / 7 decoder layers**, **fixed sine-cosine positional embeddings for visual features but learnable ones for tactile** — a small but sensible asymmetry, since tactile taxel layout has no natural spatial ordering. Predicts **50 future actions** with time aggregation for smoother output. Learning rate 1e-5 for both encoders, weight decay 1e-4.

## 3. Results

**+17.1%** average success on the UniVTAC Benchmark from adding the encoder; **+25%** in real-world robotic experiments.

**The behavioural analysis is the most useful part**, because it describes *what tactile changes about execution rather than just how often it succeeds*:

- **Insert USB** — tactile policy *"uses contact feedback to adjust alignment after initial touch, reducing misinsertion. Without tactile input, the policy often applies excessive force and fails to correct orientation, leading to jamming."*
- **Insert tube** — tactile enables *"gentle probing and real-time correction under tight clearance, avoiding damage to the fixture,"* while the vision-only policy *"tends to push harder when misaligned, causing base shift or hole deformation."* Notably: *"Recovery is possible for moderate errors with tactile, but fails under large offsets in both settings"* — a stated bound on what touch can fix.
- **Upright bottle** — vision-only shows *"noticeable jitter during lifting, likely due to delayed visual feedback and lack of slip detection."*

The summary — *"tactile sensing enhances behavioral compliance and reduces destructive interactions"* — is the recurring finding of this survey, here observed frame by frame.

## 4. What it adds that the others don't

**Infrastructure that the field actually adopted.** UniVTAC appears as the evaluation benchmark in [[n0-twam]], [[tactile-wam]], [[vitar]], [[disentvtf]] and [[ftp-1]], which makes it one of the few 2026 tactile artifacts with genuine ecosystem effects. Bundling simulator + pretrained encoder + benchmark is what enabled that: a benchmark alone would be adopted more slowly, and a simulator alone would not standardise comparison.
