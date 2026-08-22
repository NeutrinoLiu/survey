# Bibliography — *What Touch Is Worth*

111 works on tactile sensing for robot manipulation, submitted to arXiv between January and August 2026.
Each entry links its arXiv abstract page and its analysis in [`analyses/`](analyses/).
Grouped as in the report; ordered by submission date within each group.

## Benchmarks and evaluation <sub>(8)</sub>

- **TaCo (ICLR'26)** — TaCo: A Benchmark for Lossless and Lossy Codecs of Heterogeneous Tactile Data  
  [arXiv:2602.09893](https://arxiv.org/abs/2602.09893) · SJTU · Paxini · 2026-02 · [analysis](analyses/tacocodec.md)  
  ~200× compression costs ~1.5 accuracy points: touch's semantics live in few bits.

- **TacO** — TacO: Benchmarking Tactile Sensors for Object Manipulation  
  [arXiv:2605.21976](https://arxiv.org/abs/2605.21976) · UC San Diego · CMU · SNU · 2026-05 · [analysis](analyses/taco-bench.md)  
  Sensor friction changes success by 20 points with no tactile signal used.

- **roto 2.0** — roto 2.0: The Robot Tactile Olympiad  
  [arXiv:2605.21429](https://arxiv.org/abs/2605.21429) · Edinburgh · NUS · 2026-05 · [analysis](analyses/roto2.md)  
  Blind binary-contact RL reaches 13 Baoding rotations/10 s; 4× prior SOTA.

- **TacVerse** — TacVerse: A Multi-Sensor Dataset and Benchmark for Cross-Sensor Vision-Based Tactile Perception  
  [arXiv:2606.25877](https://arxiv.org/abs/2606.25877) · Imperial · QMUL · KCL · 2026-06 · [analysis](analyses/tacverse.md)  
  Cross-sensor force regression collapses to R² = −9.

- **HT-Bench** — HT-Bench: Benchmarking and Learning Dexterous Full-Hand Tactile Representations with Egocentric Vision  
  [arXiv:2606.19161](https://arxiv.org/abs/2606.19161) · Beihang · Rimbot · BUPT · 2026-06 · [analysis](analyses/ht-bench.md)  
  Four encoder-level tracks with a task-level OOD split; discrete bottleneck trades fit for transfer.

- **3D-Printable Texture Set** — A 3D-Printable Dataset for Fair Testing and Comparisons of Tactile Sensors  
  [arXiv:2606.25886](https://arxiv.org/abs/2606.25886) · Sussex · 2026-06 · [analysis](analyses/printtacbench.md)  
  A 99.5% texture classifier drops to 31% across 3D printers.

- **TactiDex** — TactiDex: A Real-World Tactile-Guided Benchmark for Human-Like Dexterous Manipulation  
  [arXiv:2607.09190](https://arxiv.org/abs/2607.09190) · ShanghaiTech · 2026-07 · [analysis](analyses/tactidex.md)  
  Tactile as RL reward, not input; a success criterion that can fail a completed task.

- **SoftVTBench** — SoftVTBench: A Deformation-Aware Visuo-Tactile Dataset and Benchmark for Deformable-Object Manipulation  
  [arXiv:2608.18701](https://arxiv.org/abs/2608.18701) · Tuojing · Tsinghua · KCL · 2026-08 · [analysis](analyses/softvtbench.md)  
  Evaluator-only FEM state: the only benchmark that can falsify a tactile gain.

## Datasets and collection systems <sub>(17)</sub>

- **TacUMI** — TacUMI: A Multi-Modal Universal Manipulation Interface for Contact-Rich Tasks  
  [arXiv:2601.14550](https://arxiv.org/abs/2601.14550) · TU Munich · Agile Robots · 2026-01 · [analysis](analyses/tacumi.md)  
  Tactile for task segmentation: skill boundaries are contact transitions.

- **DexViTac** — DexViTac: Collecting Human Visuo-Tactile-Kinematic Demonstrations for Contact-Rich Dexterous Manipulation  
  [arXiv:2603.17851](https://arxiv.org/abs/2603.17851) · HUST · 2026-03 · [analysis](analyses/dexvitac.md)  
  Local tactile is meaningless without global hand configuration.

- **VTouch++** — VTouch++: A Multimodal Dataset with Vision-Based Tactile Enhancement for Bimanual Manipulation  
  [arXiv:2604.20444](https://arxiv.org/abs/2604.20444) · Humanoid Robot (Shanghai) · 2026-04 · [analysis](analyses/vtouchpp.md)  
  Skill axes instead of task labels: recomposition without segmentation.

- **HRDexDB** — HRDexDB: A Paired Human-Robot Dataset for Cross-Embodiment Dexterous Grasping  
  [arXiv:2604.14944](https://arxiv.org/abs/2604.14944) · SNU · RLWRLD · 2026-04 · [analysis](analyses/hrdexdb.md)  
  One object set, one human hand, four robot hands, markerless.

- **TAMEn** — TAMEn: Tactile-Aware Manipulation Engine for Closed-Loop Data Collection in Contact-Rich Tasks  
  [arXiv:2604.07335](https://arxiv.org/abs/2604.07335) · Fudan · OpenDriveLab · 2026-04 · [analysis](analyses/tamen.md)  
  Recovery data collected with real tactile feedback at real failure states.

- **OmniUMI** — OmniUMI: Towards Physically Grounded Robot Learning via Human-Aligned Multimodal Interaction  
  [arXiv:2604.10647](https://arxiv.org/abs/2604.10647) · BAAI · CASIA · 2026-04 · [analysis](analyses/omniumi.md)  
  Separates tactile, internal grasp force and external wrench; bilateral feedback to the operator.

- **TouchAnything / EgoTouch** — TouchAnything: A Dataset and Framework for Bimanual Tactile Estimation from Egocentric Video  
  [arXiv:2605.13083](https://arxiv.org/abs/2605.13083) · HIT Shenzhen · Meituan · 2026-05 · [analysis](analyses/touchanything.md)  
  Wrist cameras recover the contacts the egocentric view occludes.

- **RoboTacDex** — RoboTacDex: A Dexterous Visual-Tactile-Action Dataset for Humanoid Manipulation  
  [arXiv:2606.31836](https://arxiv.org/abs/2606.31836) · Fudan · ByteDance · 2026-06 · [analysis](analyses/robotacdex.md)  
  Extra camera views buy nothing; touch buys latent state.

- **RCT** — RCT: A Robot-Collected Touch-Vision-Language Dataset for Tactile Generalization  
  [arXiv:2606.31694](https://arxiv.org/abs/2606.31694) · TU Dresden (Calandra) · 2026-06 · [analysis](analyses/rct.md)  
  Raw-pixel NN recovers 98.3% of TVL/HCT test sequences: the split is the benchmark.

- **HapTile** — HapTile: A Haptic-Informed Vision-Tactile-Language-Action Dataset for Contact-Rich Imitation Learning  
  [arXiv:2606.04825](https://arxiv.org/abs/2606.04825) · KCL · Huawei · UCL · 2026-06 · [analysis](analyses/haptile.md)  
  Haptic feedback to the operator, language as conditioning not metadata.

- **ART-Glove** — ART-Glove: Articulated Tactile Glove for Contact-Grounded Dexterous Interaction Capture  
  [arXiv:2606.16370](https://arxiv.org/abs/2606.16370) · Carnegie Mellon · 2026-06 · [analysis](analyses/art-glove.md)  
  16 rigid functional surfaces make hand-side contact geometry known, not estimated.

- **RealDexUMI** — RealDexUMI: A Wearable Universal Manipulation Interface for Dexterous Robot Learning  
  [arXiv:2606.06033](https://arxiv.org/abs/2606.06033) · Peking Univ · BeingBeyond · 2026-06 · [analysis](analyses/realdexumi.md)  
  The human wears the deployed robot hand: zero-gap tactile between collection and deployment.

- **DexTeleop-0** — DexTeleop-0: Force-Aware Bimanual Dexterous Teleoperation with Ego-Centric Perception  
  [arXiv:2606.23431](https://arxiv.org/abs/2606.23431) · NTU · OOJU · 2026-06 · [analysis](analyses/dexteleop0.md)  
  Tactile force balancing inside the teleoperation tracking loop, 56 DoF.

- **Deform360** — Deform360: A Massive Multi-view Visuotactile Dataset for Deformable World Models  
  [arXiv:2607.05390](https://arxiv.org/abs/2607.05390) · Brown · Columbia · MIT · 2026-07 · [analysis](analyses/deform360.md)  
  41 cameras + tactile over 198 deformables; 2D vs 3D world models flip by data regime.

- **PRISM** — PRISM: Precision and contact-rich Real-world Industrial Skill dataset with Multimodal sensing  
  [arXiv:2608.17962](https://arxiv.org/abs/2608.17962) · Peking Univ · Delta · 2026-08 · [analysis](analyses/prism-ind.md)  
  45 h of actual industrial assembly, not household proxies.

- **ViHaTeleop** — ViHaTeleop: A Low-Cost, Lightweight Visual-Haptic Teleoperation System for Dexterous Manipulation Learning  
  [arXiv:2608.16572](https://arxiv.org/abs/2608.16572) · Tohoku · 2026-08 · [analysis](analyses/vihateleop.md)  
  $550 haptic teleop, and a 9-participant study that reports the mixed results.

- **TWINS** — TWINS: A Tactile Wearable Isomorphic Arm Networked System for Contact-Rich Manipulation Learning  
  [arXiv:2608.01733](https://arxiv.org/abs/2608.01733) · AIST · 2026-08 · [analysis](analyses/twins.md)  
  Body-surface tactile on chest and arms; isomorphic device removes retargeting.

## Tactile world models <sub>(16)</sub>

- **VT-WM** — Visuo-Tactile World Models  
  [arXiv:2602.06001](https://arxiv.org/abs/2602.06001) · UW · FAIR Meta · 2026-02 · [analysis](analyses/vt-wm.md)  
  Named the failure: vision-only world models hallucinate contact. 33% object permanence, 29% causal compliance.

- **OmniVTA** — OmniVTA: Visuo-Tactile World Modeling for Contact-Rich Robotic Manipulation  
  [arXiv:2603.19201](https://arxiv.org/abs/2603.19201) · NUS · CASIA · TARS · 2026-03 · [analysis](analyses/omnivta.md)  
  Condition on predicted contact *change*; generated visual futures don't pay for their latency.

- **Tactile-WAM** — Tactile-WAM: Touch-Aware World Action Model with Tactile Asymmetric Attention  
  [arXiv:2606.26663](https://arxiv.org/abs/2606.26663) · Ant Group · CASIA · HKU · 2026-06 · [analysis](analyses/tactile-wam.md)  
  Tactile pollution compounds over horizon; pixel change ≠ contact change (5,000 pairs).

- **ContactWorld** — ContactWorld: What Matters in Vision-Tactile World Models for Contact-Rich Manipulation  
  [arXiv:2606.13877](https://arxiv.org/abs/2606.13877) · Purdue · Texas A&M · 2026-06 · [analysis](analyses/contactworld.md)  
  Tactile helps only when its representation is compatible with the visual one.

- **Dream-Tac** — Dream-Tac: A Unified Tactile World Action Model for Contact-Rich Robot Manipulation  
  [arXiv:2606.08737](https://arxiv.org/abs/2606.08737) · Peking Univ · HKUST · 2026-06 · [analysis](analyses/dream-tac.md)  
  A non-learned frame-difference gate, with its trace published.

- **TacForeSight** — TacForeSight: Force-Guided Tactile World Model for Contact-Rich Manipulation  
  [arXiv:2606.11184](https://arxiv.org/abs/2606.11184) · TARS · NUS · SJTU · 2026-06 · [analysis](analyses/tacforesight.md)  
  Wrist wrench precedes fingertip tactile: use one to predict the other, at 20 Hz.

- **ViTaL** — Inference-time Policy Steering via Vision and Touch  
  [arXiv:2606.14981](https://arxiv.org/abs/2606.14981) · Carnegie Mellon · 2026-06 · [analysis](analyses/vital.md)  
  Text-conditioned tactile reward; predicted latents can rank better than reality.

- **N₀-TWAM** — N0-TWAM: Scaling Tactile-Native World-Action Model for Contact-Rich Manipulation  
  [arXiv:2607.23783](https://arxiv.org/abs/2607.23783) · NeoteAI · Fudan · 2026-07 · [analysis](analyses/n0-twam.md)  
  30k h, 6 embodiments; capacity in weights, not attention masks.

- **ViTacWorld** — ViTacWorld: Scaling Visuo-Tactile World Models for Contact-Rich Robot Manipulation  
  [arXiv:2607.22530](https://arxiv.org/abs/2607.22530) · ShanghaiTech · 2026-07 · [analysis](analyses/vitacworld.md)  
  World model as data generator; tactile PSNR is mostly empty frames.

- **TouchWorld** — TouchWorld: A Predictive and Reactive Tactile Foundation Model for Dexterous Manipulation  
  [arXiv:2607.07287](https://arxiv.org/abs/2607.07287) · HIT Shenzhen · PHANES · 2026-07 · [analysis](analyses/touchworld.md)  
  1 Hz / 10 Hz / 30 Hz hierarchy; predicted subgoals beat persistence and retrieval.

- **TACO** — TACO: TActile World Model as a Self-COrrector for Scalable VLA Post-Training  
  [arXiv:2607.02840](https://arxiv.org/abs/2607.02840) · Peking Univ · AI² Robotics · 2026-07 · [analysis](analyses/taco.md)  
  Imagines the corrective demonstrations expert data structurally cannot contain.

- **FeelWorld** — FeelWorld: Visuo-Tactile World Model for Hierarchical Contact Prediction and Planning  
  [arXiv:2607.24267](https://arxiv.org/abs/2607.24267) · CASIA · Imprintx · BAAI · 2026-07 · [analysis](analyses/feelworld.md)  
  Contact / deformation / slip as a supervised hierarchy; free-space tactile degrades vision.

- **VT-WAM** — VT-WAM: Visual-Tactile World Action Model for Contact-Rich Manipulation  
  [arXiv:2607.02503](https://arxiv.org/abs/2607.02503) · CASIA · TARS · NUS · 2026-07 · [analysis](analyses/vt-wam.md)  
  Effective contact ratio is 0.26–0.41; a loss on the attention map fixes the imbalance.

- **TacWAM** — TacWAM: Anchor-Guided World Action Model with Mechanics-Aware Tactile Prediction  
  [arXiv:2607.28391](https://arxiv.org/abs/2607.28391) · Tsinghua · Manifold AI · 2026-07 · [analysis](analyses/tacwam.md)  
  Future tactile as training target only; letting the action branch read it collapses deployment.

- **HiTac-WAM** — HiTac-WAM: A Hierarchical Tactile World Action Model for Contact-Rich Robot Manipulation  
  [arXiv:2608.19574](https://arxiv.org/abs/2608.19574) · CASIA · ImprintX · BAAI · 2026-08 · [analysis](analyses/hitac-wam.md)  
  Forecast per candidate action, rank by it, then check reality against it.

- **Disentangling Visuo-Tactile Foresight** — Disentangling Visuo-Tactile Foresight: Oracle-Guided Interface Discovery for World Action Models  
  [arXiv:2608.00547](https://arxiv.org/abs/2608.00547) · BYU · BAST · 2026-08 · [analysis](analyses/disentvtf.md)  
  Oracle futures isolate the interface: even a perfect future gives only 32%.

## Tactile vision-language-action models <sub>(18)</sub>

- **TaF-VLA** — TaF-VLA: Tactile-Force Alignment in Vision-Language-Action Models for Force-aware Manipulation  
  [arXiv:2601.20321](https://arxiv.org/abs/2601.20321) · Beihang · ShanghaiTech · BIGAI · 2026-01 · [analysis](analyses/taf-vla.md)  
  Align tactile to force, not to vision; 10M pairs from an automated rig.

- **DECO (ICML'26)** — DECO: Decoupled Multimodal Diffusion Transformer for Bimanual Dexterous Manipulation with a Plugin Tactile Adapter  
  [arXiv:2602.05513](https://arxiv.org/abs/2602.05513) · XYZ Embodied AI · BAAI · TUM · 2026-02 · [analysis](analyses/deco.md)  
  Swap two modality pathways and the precision stage goes 29/40 → 0/40.

- **TacMamba** — TacMamba: A Tactile History Compression Adapter Bridging Fast Reflexes and Slow VLA Reasoning  
  [arXiv:2603.01700](https://arxiv.org/abs/2603.01700) · Zhejiang · GigaAI · 2026-03 · [analysis](analyses/tacmamba.md)  
  100 Hz tactile history in O(1); counting clicks needs memory, not readings.

- **TacVLA** — TacVLA: Contact-Aware Tactile Fusion for Robust Vision-Language-Action Manipulation  
  [arXiv:2603.12665](https://arxiv.org/abs/2603.12665) · Purdue · IIT Genova · 2026-03 · [analysis](analyses/tacvla.md)  
  Low-dimensional tactile tokens, hard-gated on contact; 2.1× under occlusion.

- **HapticVLA** — HapticVLA: Contact-Rich Manipulation via VLA Model without Inference-Time Tactile Sensing  
  [arXiv:2603.15257](https://arxiv.org/abs/2603.15257) · Skoltech · 2026-03 · [analysis](analyses/hapticvla.md)  
  Distil tactile awareness into a token; beat teachers that keep the sensor.

- **ForceVLA2** — ForceVLA2: Unleashing Hybrid Force-Position Control with Force Awareness for Contact-Rich Manipulation  
  [arXiv:2603.15169](https://arxiv.org/abs/2603.15169) · Shanghai AI Lab · SJTU · 2026-03 · [analysis](analyses/forcevla2.md)  
  Force in the action space; injecting it into the VLM pathway drops success to 5%.

- **VTAM** — VTAM: Video-Tactile-Action Models for Complex Physical Interaction Beyond VLAs  
  [arXiv:2603.23481](https://arxiv.org/abs/2603.23481) · UIUC · Stanford · SJTU · 2026-03 · [analysis](analyses/vtam.md)  
  Drop language, keep video: 0% / 0% / 10% / 90% on potato chips.

- **TacFiLM** — Tactile Modality Fusion for Vision-Language-Action Models  
  [arXiv:2603.14604](https://arxiv.org/abs/2603.14604) · McGill · Mila · NVIDIA · 2026-03 · [analysis](analyses/tacmodfusion.md)  
  FiLM beats concatenation and cross-attention; cross-attention applies 3× the force.

- **MoSS** — Modular Sensory Stream for Integrating Physical Feedback in Vision-Language-Action Models  
  [arXiv:2604.23272](https://arxiv.org/abs/2604.23272) · KAIST · RLWRLD · SNU · 2026-04 · [analysis](analyses/modsensorystream.md)  
  Existing methods degrade when given two physical modalities; MoSS is additive.

- **AT-VLA (CVPR'26)** — AT-VLA: Adaptive Tactile Injection for Enhanced Feedback Reaction in Vision-Language-Action Models  
  [arXiv:2605.07308](https://arxiv.org/abs/2605.07308) · Peking Univ · PrimeBot · 2026-05 · [analysis](analyses/at-vla.md)  
  Three tactile formats × two injection strategies: direct injection is worse than no tactile.

- **UniTacVLA** — UniTacVLA: Unified Tactile Understanding and Prediction in Vision Language Action Models  
  [arXiv:2606.31723](https://arxiv.org/abs/2606.31723) · HIT · Daimon Robotics · 2026-06 · [analysis](analyses/unitacvla.md)  
  Tactile chain-of-thought: the contact reasoning is written out in language.

- **TAP-VLA** — TAP-VLA: Tactile Annotation Prompting for Vision Language Action Models  
  [arXiv:2606.29089](https://arxiv.org/abs/2606.29089) · Univ. of Michigan · 2026-06 · [analysis](analyses/tap-vla.md)  
  Draw the shear field on the camera image: presentation dominates signal.

- **TORL-VLA** — TORL-VLA: Tactile Guided Online Reinforcement Learning for Contact-Rich Manipulation  
  [arXiv:2606.09337](https://arxiv.org/abs/2606.09337) · Meituan · BIT · 2026-06 · [analysis](analyses/torl-vla.md)  
  Contact awareness ≠ contact adaptation; intervention-censored critic fixes credit assignment.

- **TacCoRL** — TacCoRL: Integrating Tactile Feedback into VLA via Simulation  
  [arXiv:2606.11743](https://arxiv.org/abs/2606.11743) · UCLA · UCSD · PKU · 2026-06 · [analysis](analyses/taccorl.md)  
  Near-failure states are rare in demos and unsafe on hardware: learn them in sim.

- **N₀-VTLA** — N0-VTLA: Scaling Vision-Tactile-Language-Action Model with Latent Tactile Tokens  
  [arXiv:2607.23782](https://arxiv.org/abs/2607.23782) · NeoteAI · Fudan · 2026-07 · [analysis](analyses/n0-vtla.md)  
  Touch belongs in the action pathway as a prediction, not in the language prefix.

- **τ** — tau: Learning Touch-Augmented Vision-Language-Action Models from Future Visual Supervision  
  [arXiv:2607.24485](https://arxiv.org/abs/2607.24485) · BJTU · BIGAI · 2026-07 · [analysis](analyses/tau-vla.md)  
  Supervise the tactile encoder with future *visual* feature change.

- **ResTacVLA** — Feeling the Unexpected: ResTacVLA for Contact-Rich Manipulation via Residual Tactile Representation  
  [arXiv:2607.03387](https://arxiv.org/abs/2607.03387) · UCAS · CASIA · Dexmal · 2026-07 · [analysis](analyses/restacvla.md)  
  Predictive coding: feed the policy only the part of touch vision failed to predict.

- **ViTaR** — ViTaR: Visuo-Tactile Residual Adaptation for Foundation VLA Manipulation  
  [arXiv:2608.15816](https://arxiv.org/abs/2608.15816) · Beijing Inst. of Technology · 2026-08 · [analysis](analyses/vitar.md)  
  Tactile as bounded execution modulator; 0.04 unsafe rate vs 0.25–0.31 for RL.

## Tactile representation and cross-sensor transfer <sub>(13)</sub>

- **AnyTouch 2** — AnyTouch 2: General Optical Tactile Representation Learning For Dynamic Tactile Perception  
  [arXiv:2602.09617](https://arxiv.org/abs/2602.09617) · Renmin · BAAI · 2026-02 · [analysis](analyses/anytouch2.md)  
  A five-tier dynamic pyramid, and a measured static-vs-dynamic trade-off in alignment.

- **UniForce** — UniForce: A Unified Latent Force Model for Robot Manipulation with Diverse Tactile Sensors  
  [arXiv:2602.01153](https://arxiv.org/abs/2602.01153) · KCL · Imperial · UCL · Bristol · 2026-02 · [analysis](analyses/uniforce.md)  
  Force equilibrium in a two-finger grasp is a free cross-sensor label.

- **TactAlign** — TactAlign: Human-to-Robot Policy Transfer via Tactile Alignment  
  [arXiv:2602.13579](https://arxiv.org/abs/2602.13579) · Michigan · NVIDIA · Microsoft · 2026-02 · [analysis](analyses/tactalign.md)  
  Unpaired alignment by rectified flow: strict pairing dies under sliding contact.

- **HTT** — Heterogeneous Tactile Transformer  
  [arXiv:2606.29948](https://arxiv.org/abs/2606.29948) · NUS · CMU · 2026-06 · [analysis](analyses/htt.md)  
  1.6M optical↔array paired frames; spatial layout is what a wrench cannot carry.

- **UniTac** — UniTac: A Unified Multimodal Model for Cross-Sensor Tactile Understanding and Generation  
  [arXiv:2606.31451](https://arxiv.org/abs/2606.31451) · Zhejiang · Yale · Oxford · MIT · 2026-06 · [analysis](analyses/unitac.md)  
  Sensor level and object level are two stages of one acquisition; generation closes a 50%→99% gap.

- **TactX** — TactX: Learning Shared Tactile Representations Across Diverse Sensors  
  [arXiv:2606.31236](https://arxiv.org/abs/2606.31236) · UC San Diego · SNU · Amazon FAR · 2026-06 · [analysis](analyses/tactx.md)  
  Resistive, magnetic, vision-based in one latent, paired by different-sensor-per-finger.

- **FTP-1** — FTP-1: A Generalist Foundation Tactile Policy Across Tactile Sensors  
  [arXiv:2606.13102](https://arxiv.org/abs/2606.13102) · Tsinghua · Sharpa · SJTU · 2026-06 · [analysis](analyses/ftp-1.md)  
  Function, not format: a gripper's pads are a thumb and an index fingertip.

- **TacGen** — TacGen: Touch Is a Necessary Dimension of Physical-World Representation  
  [arXiv:2606.29173](https://arxiv.org/abs/2606.29173) · Maryland · CMU · Stanford · 2026-06 · [analysis](analyses/tacgen.md)  
  Vision-only capacity scaling recovers 4.5% of the gap; touch is necessary evidence.

- **TTP** — Human-Centric Transferable Tactile Pre-Training for Dexterous Robotic Manipulation  
  [arXiv:2607.01067](https://arxiv.org/abs/2607.01067) · Peking Univ · BeingBeyond · 2026-07 · [analysis](analyses/hcttp.md)  
  160 h of human tactile, with pre- and post-training held strictly consistent.

- **FELT** — FELT: Generating Tactile Signals from Vision for Visuo-Tactile Manipulation  
  [arXiv:2607.20683](https://arxiv.org/abs/2607.20683) · USC · Columbia · Starpilot · 2026-07 · [analysis](analyses/felt.md)  
  RGB→tactile at 20 ms, deployed closed-loop without a sensor.

- **VQ-Touch** — VQ-Touch: A Data-Efficient Tactile Generation Framework Across Sensors and Scenarios  
  [arXiv:2607.14728](https://arxiv.org/abs/2607.14728) · CASIA · Zhongguancun · 2026-07 · [analysis](analyses/vq-touch.md)  
  Model sensors at the family level; transfer within a family from few shots.

- **RATG** — Representation-Aligned Tactile Grounding for Contact-Rich Robotic Manipulation  
  [arXiv:2607.14609](https://arxiv.org/abs/2607.14609) · Fudan · Lenovo · NTU · 2026-07 · [analysis](analyses/ratg.md)  
  Probe where future tactile is linearly accessible; supervising everywhere is worse than nowhere.

- **EgoTac** — EgoTac: In-the-wild Tactile Prediction from Egocentric Vision  
  [arXiv:2608.15060](https://arxiv.org/abs/2608.15060) · Shanghai Qi Zhi · SJTU · 2026-08 · [analysis](analyses/egotac.md)  
  Label existing egocentric video with touch retroactively; both scaling axes still climbing.

## Tactile language models and reasoning <sub>(7)</sub>

- **RobustTouch** — Test-Time Adaptation for Tactile-Vision-Language Models  
  [arXiv:2602.15873](https://arxiv.org/abs/2602.15873) · Shenzhen Tech · NYU · 2026-01 · [analysis](analyses/tta-tvl.md)  
  Asynchronous shift: the tactile sensor is the one that wears out.

- **FG-CLTP** — FG-CLTP: Fine-Grained Contrastive Language Tactile Pretraining for Robotic Manipulation  
  [arXiv:2603.10871](https://arxiv.org/abs/2603.10871) · CASIA · BAAI · 2026-03 · [analysis](analyses/fg-cltp.md)  
  Adjectives cannot say 5 N vs 20 N: tokenise the numbers.

- **VitaTouch** — VitaTouch: Property-Aware Vision-Tactile-Language Model for Robotic Quality Inspection  
  [arXiv:2604.03322](https://arxiv.org/abs/2604.03322) · BUPT · Zhongguancun · BIT · 2026-04 · [analysis](analyses/vitatouch.md)  
  Touch for industrial inspection: less illumination-sensitive than vision on glossy metal.

- **Touch-R1** — Touch-R1: Reinforcing Touch Reasoning in MLLMs  
  [arXiv:2605.27154](https://arxiv.org/abs/2605.27154) · Xiamen · Daimon Robotics · 2026-05 · [analysis](analyses/touch-r1.md)  
  Reward credit only when real tactile beats ablated-input counterfactuals.

- **TouchThinker** — TouchThinker: Scaling Tactile Commonsense Reasoning to the Open World  
  [arXiv:2606.11637](https://arxiv.org/abs/2606.11637) · CASIA · NUS · 2026-06 · [analysis](analyses/touchthinker.md)  
  Tactile is action-specific: press reveals hardness, slide reveals friction.

- **GeoTLM** — GeoTLM: Geometry-aware Tactile-Language Models for Contact Motion Orientation Reasoning  
  [arXiv:2606.15909](https://arxiv.org/abs/2606.15909) · Nanyang Technological Univ · 2026-06 · [analysis](analyses/geotlm.md)  
  Sparsh and AnyTouch2 are at chance on rotation direction; 14K parameters fix it.

- **Splash** — Wake up for Touch! Mask-isolated Tactile Alignment Learning in MLLMs  
  [arXiv:2607.00302](https://arxiv.org/abs/2607.00302) · Ewha Womans University · 2026-07 · [analysis](analyses/waketouch.md)  
  Confine tactile learning to dormant parameters; measure what it costs on VL tasks.

## Simulation and sim-to-real <sub>(12)</sub>

- **UniVTAC** — UniVTAC: A Unified Simulation Platform for Visuo-Tactile Manipulation Data Generation, Learning, and Benchmarking  
  [arXiv:2602.10093](https://arxiv.org/abs/2602.10093) · ScaleLab SJTU · D-Robotics · 2026-02 · [analysis](analyses/univtac.md)  
  Simulator + encoder + benchmark, and the field adopted all three.

- **Tacmap** — Tacmap: Bridging the Tactile Sim-to-Real Gap via Geometry-Consistent Penetration Depth Map  
  [arXiv:2602.21625](https://arxiv.org/abs/2602.21625) · Sharpa · HKUST · NVIDIA · 2026-02 · [analysis](analyses/tacmap.md)  
  Align sim and real in deformation space, not pixel space; curved gels supported.

- **HydroShear** — HydroShear: Hydroelastic Shear Simulation for Tactile Sim-to-Real Reinforcement Learning  
  [arXiv:2603.00446](https://arxiv.org/abs/2603.00446) · Michigan · Amazon Robotics · 2026-02 · [analysis](analyses/hydroshear.md)  
  Policies trained on simulated shear reach 93%; on simulated images, 34%.

- **Tac2Real** — Tac2Real: Reliable and GPU Visuotactile Simulation for Online RL and Zero-Shot Real-World Deployment  
  [arXiv:2603.28475](https://arxiv.org/abs/2603.28475) · Shanghai AI Lab · HKUST · 2026-03 · [analysis](analyses/tac2real.md)  
  Structured gap gets calibrated, stochastic gap gets randomised.

- **PTLD** — PTLD: Sim-to-real Privileged Tactile Latent Distillation for Dexterous Manipulation  
  [arXiv:2603.04531](https://arxiv.org/abs/2603.04531) · CMU · UW · FAIR · Berkeley · 2026-03 · [analysis](analyses/ptld.md)  
  Distil real touch into a privileged policy's latent: never simulate tactile.

- **ETac** — ETac: A Lightweight and Efficient Tactile Simulation Framework for Learning Dexterous Manipulation  
  [arXiv:2604.20295](https://arxiv.org/abs/2604.20295) · ShanghaiTech · 2026-04 · [analysis](analyses/etac.md)  
  Learned deformation propagation: FEM-comparable at 4,096 parallel environments.

- **DOT-Sim** — DOT-Sim: Differentiable Optical Tactile Simulation with Precise Real-to-Sim Physical Calibration  
  [arXiv:2604.27367](https://arxiv.org/abs/2604.27367) · Stanford · Cambridge · 2026-04 · [analysis](analyses/dot-sim.md)  
  Differentiable MPM calibrated in minutes; optics as a residual to the idle state.

- **IsaacIPC** — IsaacIPC: Coupling High-Fidelity Simulation and Realistic Rendering for Contact-Rich Robotic Systems  
  [arXiv:2605.24339](https://arxiv.org/abs/2605.24339) · Anker Humanoid Lab · HKU · 2026-05 · [analysis](analyses/isaacipc.md)  
  Contact-pressure accuracy as an explicit target; normal only, shear acknowledged missing.

- **Tactile Genesis** — Tactile Genesis: Exploring Tactile Sensors at Scale for Learning Dexterous Tasks  
  [arXiv:2606.22332](https://arxiv.org/abs/2606.22332) · Carnegie Mellon · Genesis AI · 2026-06 · [analysis](analyses/tactilegenesis.md)  
  Placement dominates sensor type; 200 taxels across the whole hand suffices.

- **TaCauchy (IROS'26)** — TaCauchy: An Extensible FEM Framework for Vision-Based Tactile Simulation  
  [arXiv:2606.20426](https://arxiv.org/abs/2606.20426) · Tsinghua SIGS · Huawei · 2026-06 · [analysis](analyses/tacauchy.md)  
  Cauchy stress from constitutive law, <1 ms extraction overhead.

- **TactSpace** — TactSpace: Learning a Physics-enriched Shared Latent Space for Tactile Sim-to-Real Transfer  
  [arXiv:2606.18959](https://arxiv.org/abs/2606.18959) · ETH Zürich · NVIDIA · 2026-06 · [analysis](analyses/tactspace.md)  
  Align simulated depth with real capacitance in latent space; skip signal fidelity.

- **SBLR** — Tactile Sim2Real without Tactile Simulation via Bottlenecked Latent Reconstruction  
  [arXiv:2608.15897](https://arxiv.org/abs/2608.15897) · Michigan · Google DeepMind · 2026-08 · [analysis](analyses/tacblr.md)  
  Unpaired random play beats a physics-based tactile simulator by 7.5–15 points.

## Policies and control <sub>(12)</sub>

- **CGP** — Contact-Grounded Policy: Dexterous Visuotactile Policy with Generative Contact Grounding  
  [arXiv:2603.05687](https://arxiv.org/abs/2603.05687) · Purdue · Meta Reality Labs · 2026-03 · [analysis](analyses/cgp.md)  
  The compliance controller's tracking error *is* a contact signal.

- **HTD** — Learning Versatile Humanoid Manipulation with Touch Dreaming  
  [arXiv:2604.13015](https://arxiv.org/abs/2604.13015) · CMU · Bosch Center for AI · 2026-04 · [analysis](analyses/touchdreaming.md)  
  Touch dreaming inside behaviour cloning; latent targets beat raw by 30% relative.

- **TDP** — Tube Diffusion Policy: Reactive Visual-Tactile Policy Learning for Contact-rich Manipulation  
  [arXiv:2604.23609](https://arxiv.org/abs/2604.23609) · Meta Reality Labs · EPFL · 2026-04 · [analysis](analyses/tubedp.md)  
  Action chunking is the obstacle: replace the trajectory with a tube around it.

- **T-Rex** — T-Rex: Tactile-Reactive Dexterous Manipulation  
  [arXiv:2606.17055](https://arxiv.org/abs/2606.17055) · UC Berkeley · NVIDIA · Stanford · 2026-06 · [analysis](analyses/t-rex.md)  
  Tactile can be acquired in mid-training; 22,889 h of human video needs no touch.

- **VibeAct** — VibeAct: Vibration to Actions for Contact-Rich Reactive Robot Dexterity  
  [arXiv:2606.27344](https://arxiv.org/abs/2606.27344) · CMU · Bosch Center for AI · 2026-06 · [analysis](analyses/vibeact.md)  
  Microphones as tactile; contact and slip as the representation both domains can produce.

- **Blind Dexterous Grasping** — Blind Dexterous Grasping via Real2Sim2Real Tactile Policy Learning  
  [arXiv:2606.11767](https://arxiv.org/abs/2606.11767) · ShanghaiTech · BIGAI · 2026-06 · [analysis](analyses/blinddexgrasp.md)  
  No vision at all; 27% on 20 objects, reported honestly.

- **MiTaS** — Multi-Resolution Tactile Imitation Learning for Contact-Rich Robotic Manipulation  
  [arXiv:2606.06281](https://arxiv.org/abs/2606.06281) · TU Darmstadt · Hessian AI · 2026-06 · [analysis](analyses/mitas.md)  
  Spatial and high-frequency tactile are different modalities; co-training transfers.

- **RGB-S** — RGB-S: Image-Aligned Tactile Saliency for Robust Dexterous Manipulation  
  [arXiv:2606.08765](https://arxiv.org/abs/2606.08765) · ShanghaiTech · BIGAI · 2026-06 · [analysis](analyses/rgb-s.md)  
  Project contacts into the image with forward kinematics: the correspondence is known.

- **WT-UMI** — WT-UMI: Tactile-based Whole-Body Manipulation via Force-Supervised Contact-Aware Planning  
  [arXiv:2606.13232](https://arxiv.org/abs/2606.13232) · Georgia Tech · 2026-06 · [analysis](analyses/wt-umi.md)  
  Predict a contact-force trajectory and give it to an admittance controller.

- **OmniTacTune** — OmniTacTune: Policy-Agnostic Real-World RL for Tactile Residual Adaptation of Visual Policies  
  [arXiv:2607.03723](https://arxiv.org/abs/2607.03723) · Maryland · Georgia Tech · 2026-07 · [analysis](analyses/omnitactune.md)  
  5–40% → 85–100% in 40–80 minutes of real-world RL, any base policy.

- **ReTouch** — ReTouch: Empowering Contact-Rich Dexterous Manipulation with Online-Refined Tactile Prediction  
  [arXiv:2608.01824](https://arxiv.org/abs/2608.01824) · USTC · iFLYTEK · CUHK · 2026-08 · [analysis](analyses/retouch.md)  
  A fixed one-shot tactile forecast is worth 0.8 points; refresh it mid-chunk.

- **CAAT** — CAAT: Contact-Aware Attention Scaling and Tactile Masking for Data-Efficient Contact-Rich Manipulation  
  [arXiv:2608.01102](https://arxiv.org/abs/2608.01102) · ShanghaiTech · Beihang · BIGAI · 2026-08 · [analysis](analyses/caat.md)  
  An explicit contact prior beats a learned gate, most in the low-data regime.

## Sensors and systems <sub>(5)</sub>

- **SpikingTac** — SpikingTac: A Miniaturized Neuromorphic Visuotactile Sensor  
  [arXiv:2602.23654](https://arxiv.org/abs/2602.23654) · CASIA · UCAS · 2026-02 · [analysis](analyses/spikingtac.md)  
  Event tactile under $150, with the elastomer hysteresis actually solved.

- **FingerEye** — FingerEye: Learning Dexterous Manipulation with Continuous Vision-Tactile Sensing  
  [arXiv:2604.20689](https://arxiv.org/abs/2604.20689) · NUS · RoboScience · 2026-04 · [analysis](analyses/fingereye.md)  
  One fingertip that works before and after contact; group fusion against modality shortcuts.

- **TransTac** — TransTac: Visuo-Tactile Modality Transition via Ultraviolet-Encoded Transparent Elastomers  
  [arXiv:2606.04477](https://arxiv.org/abs/2606.04477) · BUPT · 2026-06 · [analysis](analyses/transtac.md)  
  A transparent gel whose tactile images a VLM can already read: 83.3% zero-shot.

- **Conformal EIT Skin** — Toward Geometry-Scalable Whole-Body Touch for Humanoids: A 3D-Printed Conformal EIT Skin  
  [arXiv:2608.02080](https://arxiv.org/abs/2608.02080) · CTU Prague · Colorado · KAIST · 2026-08 · [analysis](analyses/eitskin.md)  
  3D-print a continuous sensing domain onto any curvature; 6 mm on curved surfaces.

- **Near-sensor Computing** — Near-sensor Computing for Rapid Visuotactile Perception  
  [arXiv:2608.05725](https://arxiv.org/abs/2608.05725) · ShanghaiTech · 2026-08 · [analysis](analyses/nearsensor.md)  
  Poisson solve on FPGA: reflex loop 170 ms → 28 ms, deterministic.

## Surveys <sub>(3)</sub>

- **Tactile Multimodal Fusion Survey** — Tactile-based Multimodal Fusion in Embodied Intelligence: A Survey  
  [arXiv:2605.17336](https://arxiv.org/abs/2605.17336) · XJTU · HKUST(GZ) · 2026-05 · [analysis](analyses/tacfusionsurvey.md)  
  First dedicated survey of multimodal tactile fusion; documents the evaluation fragmentation.

- **Vision-Based Tactile Intelligence** — Vision-Based Tactile Intelligence for Robotics: Sensing, Learning, and Embodied Manipulation  
  [arXiv:2608.15490](https://arxiv.org/abs/2608.15490) · Great Bay · Tsinghua · HKU · KCL · 2026-08 · [analysis](analyses/vbtintel.md)  
  Hardware, learning, simulation and datasets as one sensing-and-learning system.

- **Learning Physical Interaction** — Learning Physical Interaction: A Survey of Tactile- and Force-aware Robot Learning  
  [arXiv:2608.07558](https://arxiv.org/abs/2608.07558) · NTU · Stanford · Berkeley · MIT · 2026-08 · [analysis](analyses/physintsurvey.md)  
  TF-ART phases; compliance can be hardware, and body-policy co-design follows.
