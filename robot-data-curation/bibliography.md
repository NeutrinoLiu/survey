# Data Curation & Cleaning for Robotics Foundation Model Pretraining — Annotated Bibliography (2025–2026)

Survey compiled 2026-08-21. Scope: works released 2025 or later (heavily weighted to 2026) that are a
robotics/embodied foundation model — or the pretraining corpus behind one — **and** document a data
curation / cleaning / filtering pipeline, ideally with data-scaling evidence.

Every entry links to its per-work analysis in `./analyses/`. The downloaded source artifacts
(PDF / HTML / README) are archived outside this repository.
The **work title links to the original arXiv page, technical report, or project site**;
the slug in the first column links to the local analysis.
Overview report: **[index.html](index.html)**

**Disclosure level** grades how much of the pipeline is actually published:
`A+` exhaustive with numeric thresholds · `A` full paper · `B` blog/tech-report with real detail ·
`C` press release or trade coverage · `D` essentially nothing.

---

## 1. Human-video corpora and the scaling-law works

| Slug | Work | Org | Date | Scale | Disc. |
|---|---|---|---|---|---|
| [`dyna-2`](analyses/dyna-2.md) | [DYNA-2: A 1-Million-Hour Scaling Law for World-Action Models](https://www.dyna.co/dyna-2) | Dyna Robotics | 2026-08 | **1,000,000 h** human ego video, zero robot data in pretraining | B |
| [`egoscale`](analyses/egoscale.md) | [EgoScale: Scaling Dexterous Manipulation with Diverse Egocentric Human Data](https://arxiv.org/abs/2602.16710) | UT Austin RPL + | 2026-02 | **20,854 h**; fitted law `L = 0.024 − 0.003·ln(D)`, R²=0.9983 | A |
| [`humannet`](analyses/humannet.md) | [HumanNet: Scaling Human-centric Video Learning to One Million Hours](https://arxiv.org/abs/2605.06747) | PKU DAGroup | 2026-05 | **1,000,000 h**, 967K videos; robot-ready gate <15 mm / >60% | A |
| [`egoverse`](analyses/egoverse.md) | [EgoVerse: An Egocentric Human Dataset for Robot Learning from Around the World](https://arxiv.org/abs/2604.07607) | GT/Stanford/UCSD/ETH/MIT/Meta | 2026-04 | 1,362 h · 1,965 tasks · 240 scenes · 2,087 demonstrators | A |
| [`being-h0.7`](analyses/being-h0.7.md) | [Being-H0.7: A Latent World-Action Model from Egocentric Videos](https://arxiv.org/abs/2605.00078) | BeingBeyond | 2026-05 | 200,000 h human + 15,000 h robot | A |
| [`open-aoe`](analyses/open-aoe.md) | [Open-AoE: An Open Egocentric Manipulation Dataset and Toolchain](https://arxiv.org/abs/2607.14183) | Ant Research | 2026-07 | ~2,000 h, 500+ contributors, 400+ phone models | A |
| [`ace-ego-0`](analyses/ace-ego-0.md) | [ACE-Ego-0: Unifying Egocentric Human and Robotic Data for VLA Pretraining](https://arxiv.org/abs/2606.17200) | ACE | 2026-06 | 1.48K h pseudo-labelled human + 4.53K h robot/sim | **A+** |
| [`physbrain-1.0`](analyses/physbrain-1.0.md) | [PhysBrain 1.0 Technical Report](https://arxiv.org/abs/2605.15298) | DeepWisdom | 2026-05 | ~1,000 h → structured physical-commonsense QA | A |
| [`psi-0`](analyses/psi-0.md) | [Ψ₀: An Open Foundation Model Towards Universal Humanoid Loco-Manipulation](https://arxiv.org/abs/2603.12263) | Physical Superintelligence Lab | 2026-03 | ~800 h EgoDex + ~30 h robot — **the purity-over-volume counterexample** | A |
| [`figure-helix-02`](analyses/figure-helix-02.md) | [Helix 02 + Project Go-Big](https://www.figure.ai/news/project-go-big) | Figure AI | 2025-09 / 2026-01 | Brookfield 100,000+ residences; zero robot demos for navigation | C |

## 2. Frontier VLA / foundation models with documented mixtures

| Slug | Work | Org | Date | Data note | Disc. |
|---|---|---|---|---|---|
| [`qwen-robotmanip`](analyses/qwen-robotmanip.md) | [Qwen-RobotManip Technical Report: Alignment Unlocks Scale](https://arxiv.org/abs/2606.17846) | Alibaba Qwen | 2026-06 | **~38,100 h**, open data only; 5-stage filter + 3 cross-modal checks | **A+** |
| [`qwen-vla`](analyses/qwen-vla.md) | [Qwen-VLA: Unifying VLA across Tasks, Environments, Embodiments](https://arxiv.org/abs/2605.30280) | Alibaba Qwen | 2026-05 | Published mixture table (74.2% robot trajectories) | A |
| [`pi-0.7`](analyses/pi-0.7.md) | [π₀.₇: a Steerable Generalist Robotic Foundation Model](https://arxiv.org/abs/2604.15483) | Physical Intelligence | 2026-04 | **Condition on quality instead of filtering it** | A |
| [`pi-star-0.6-recap`](analyses/pi-star-0.6-recap.md) | [π*₀.₆ / RECAP: a VLA that Learns from Experience](https://arxiv.org/abs/2511.14759) | Physical Intelligence | 2025-11 | Learned value function grades every trajectory | A |
| [`pi-0.5`](analyses/pi-0.5.md) | [π₀.₅: a VLA with Open-World Generalization](https://arxiv.org/abs/2504.16054) | Physical Intelligence | 2025-04 | Co-training across abstraction levels; hybrid multi-modal examples | A |
| [`generalist-gen-1`](analyses/generalist-gen-1.md) | [GEN-0 / GEN-1: Embodied Foundation Models That Scale](https://generalistai.com/blog/gen-1) | Generalist AI | 2025-11 / 2026-04 | **270K → 500K h**, +10K h/week; power law `L(D)=(D_c/D)^α`; ossification below 7B | B |
| [`groot-n1`](analyses/groot-n1.md) · [`groot-n1.7`](analyses/groot-n1.7.md) | [GR00T N1](https://arxiv.org/abs/2503.14734) → [N1.7](https://huggingface.co/nvidia/GR00T-H-N1.7) | NVIDIA | 2025-03 → 2026 | **Data pyramid**; 88 h → 827 h neural augmentation; data released | A / B |
| [`gemini-robotics-2`](analyses/gemini-robotics-2.md) | [Gemini Robotics 2 / ER 2 / On-Device 2](https://deepmind.google/blog/gemini-robotics-2-brings-whole-body-intelligence-to-robots/) | Google DeepMind | 2026-07 | ⚠️ Model card defers entirely to Gemini 3.5 Flash | **D** |
| [`gr-3`](analyses/gr-3.md) · [`bytedance-gr-rl`](analyses/bytedance-gr-rl.md) | [GR-3](https://arxiv.org/abs/2507.15493) → [GR-RL](https://seed.bytedance.com/en/direction/robotics) | ByteDance Seed | 2025-07 → 2026 | Collection **scheduler**; invalid-task refusal eval; shoe-lacing 45.7→83.3% | A / C |
| [`abot-m0`](analyses/abot-m0.md) | [ABot-M0 (UniACT-dataset)](https://arxiv.org/abs/2602.11236) | Alibaba AMAP | 2026-02 | **6M+ trajectories / 9,500 h** from 6 public datasets; Gini/Lorenz balancing | A |
| [`galaxea-g0`](analyses/galaxea-g0.md) | [Galaxea Open-World Dataset + G0 → G0.5](https://arxiv.org/abs/2509.00576) | Galaxea | 2025-09 → 2026-01 | 500 h single-embodiment; **cross-embodiment pretraining can degrade** | A |
| [`hy-embodied-0.5`](analyses/hy-embodied-0.5.md) · [`hy-embodied-vla-0.5`](analyses/hy-embodied-vla-0.5.md) | [HY-Embodied-0.5](https://arxiv.org/abs/2604.07430) / [VLA](https://arxiv.org/abs/2606.14409) / RxBrain 1.0 | Tencent Hunyuan | 2026-04 → 07 | **Taxonomy-first curation**; >100M samples; >10,000 h action data | A |
| [`ace-brain-0.5`](analyses/ace-brain-0.5.md) | [ACE-Brain-0.5: Unified Embodied Foundational Model](https://arxiv.org/abs/2607.04426) | ACE-Brain Team | 2026-07 | **Cross-interface interference**; train-merge-reactivate | A |
| [`wall-oss-0.5`](analyses/wall-oss-0.5.md) | [Wall-OSS-0.5 (Gradient-Bridged Pretraining) · WALL-WM](https://github.com/X-Square-Robot/wall-x) | X Square Robot | 2026-05 | 3-source mixture + 90M embodied bridge samples; single-stage | B |
| [`robobrain-2.5`](analyses/robobrain-2.5.md) | [RoboBrain 2.5: Depth in Sight, Time in Mind](https://arxiv.org/abs/2601.14352) | BAAI | 2026-01 | `(u,v,d)` labels chosen for **2D-dataset reusability** | A |
| [`spirit-v1.5`](analyses/spirit-v1.5.md) | [Spirit-v1.5](https://github.com/Spirit-AI-Team/spirit-v1.5) | Spirit AI | 2026-01 | #1 RoboChallenge Table30; dataset/transform code released | B |
| [`agibot-go-2`](analyses/agibot-go-2.md) | [AGIBOT GO-2 + AGIBOT WORLD 2026](https://www.agibot.com/article/231/detail/63.html) | AGIBOT | 2026-04/08 | Free-form collection; **error-recovery trajectories retained & annotated** | C |
| [`skild-brain`](analyses/skild-brain.md) | [Skild Brain (omni-bodied)](https://www.skild.ai/blogs/omni-bodied) | Skild AI | 2025-09 → 2026 | **100,000 procedurally generated robot bodies** | C |
| [`1x-redwood`](analyses/1x-redwood.md) | [Redwood + 1X World Model Lab](https://www.1x.tech/discover/1x-world-model-lab) | 1X Technologies | 2025 → 2026-06 | Explicit 5-source mixture doctrine; on-policy NEO flywheel | C |
| [`lap`](analyses/lap.md) | [LAP: Language-Action Pre-Training](https://arxiv.org/abs/2602.10556) | Princeton + Physical Intelligence | 2026-02 | Actions as **natural language** — deletes the alignment pipeline | A |

## 3. World models & video-action models

| Slug | Work | Org | Date | Data note | Disc. |
|---|---|---|---|---|---|
| [`hydra-0`](analyses/hydra-0.md) | [Hydra-0: Action Flow for Generalist World Modeling and Control](https://arxiv.org/abs/2608.18077) | NVIDIA | 2026-08 | **Publishes per-source before/after counts** — DROID loses 54.9% of windows | **A+** |
| [`cosmos-3`](analyses/cosmos-3.md) | [Cosmos 3: Omnimodal World Models for Physical AI](https://research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf) | NVIDIA | 2026-06 | **7.8B → 767M images (9.8% yield)**; SILA infrastructure | **A+** |
| [`cosmos-wfm-physical-ai`](analyses/cosmos-wfm-physical-ai.md) | [Cosmos-Predict2.5 / Transfer2.5](https://arxiv.org/abs/2511.00062) | NVIDIA | 2026-02 | 200M curated clips; Sim2Real/Real2Real translation | A |
| [`cosmos-policy`](analyses/cosmos-policy.md) | [Cosmos Policy: Fine-Tuning Video Models for Visuomotor Control](https://arxiv.org/abs/2601.16163) | NVIDIA + Stanford | 2026-01 | Curation paid once upstream; actions as latent frames | A |
| [`kairos`](analyses/kairos.md) | [Kairos: A Regret-Aware Native World-Action Model Stack](https://arxiv.org/abs/2606.16533) | Kairos Team | 2026-07 | **Quality vs control-relevance**; every filter named with its model | **A+** |
| [`abot-world-0`](analyses/abot-world-0.md) | [ABot-World-0: Infinite Interactive World Rollout](https://arxiv.org/abs/2607.19191) | Alibaba AMAP | 2026-07 | **14 checks / 6 dimensions**, hard-reject vs soft-flag | **A+** |
| [`lingbot-va-2`](analyses/lingbot-va-2.md) | [Native Video-Action Pretraining (LingBot-VA 2.0)](https://arxiv.org/abs/2607.08639) | Ant Group Robbyant | 2026-07 | **Robot→human** synthesis; whole-finger-envelope gripper aperture | A |
| [`dreamzero`](analyses/dreamzero.md) | [World Action Models are Zero-shot Policies (DreamZero)](https://arxiv.org/abs/2602.15922) | NVIDIA GEAR | 2026-02 | 500 h / 22 envs; **diversity beats repetition at matched hours** | A |
| [`cortex-2.0`](analyses/cortex-2.0.md) | [Cortex 2.0: Grounding World Models in Industrial Deployment](https://arxiv.org/abs/2604.20246) | Sereact GmbH | 2026-04 | Test-time candidate scoring; industrial clutter/occlusion | B |
| [`mimic-flux`](analyses/mimic-flux.md) | [FLUX-mimic / mimic-video](https://github.com/mimic-video/mimic-video) | mimic robotics + Black Forest Labs | 2026-07 | 30 min robot data per task (vs 30+ h) | C |

## 4. Data engines, synthetic pipelines, benchmarks

| Slug | Work | Org | Date | Data note | Disc. |
|---|---|---|---|---|---|
| [`axis`](analyses/axis.md) | [AXIS: A Growable Community-Driven Data Engine](https://arxiv.org/abs/2607.21588) | — | 2026-07 | 207 tasks / 50K+ traj; **success checker generated with the task** | A |
| [`robocurate`](analyses/robocurate.md) | [RoboCurate: Action-Verified Neural Trajectory](https://arxiv.org/abs/2602.18742) | KAIST | 2026-02 | **Simulator replay verifies the action label** | A |
| [`robocasa365`](analyses/robocasa365.md) | [RoboCasa365: Large-Scale Simulation Framework](https://arxiv.org/abs/2603.04356) | — | 2026-03 | 300+ tasks / 2,500 scenes / 500K+ demos | A |
| [`vla-data-survey`](analyses/vla-data-survey.md) | [VLA in Robotics: Survey of Datasets, Benchmarks, and Data Engines](https://arxiv.org/abs/2604.23001) | UMD + Utah | 2026-04 | The taxonomy this survey uses for framing | A |

---

## Cross-cutting findings

**1. Three answers to "what do we do about bad data?"**
- **Filter it out** — Qwen-RobotManip (5-stage cascade, 81% of RoboMIND UR episodes rejected), Cosmos 3 (9.8% image yield), Hydra-0 (16.7% of windows), ABot-World-0 (14 checks).
- **Condition on it** — π₀.₇ (human quality 1–5 + mistake flags as prompt fields), RECAP (learned advantage), AGIBOT (annotated error-recovery trajectories).
- **Route it to a weaker objective** — DYNA-2 (episodes failing the hand-pose bar feed video-prediction only), ACE-Ego-0 (reliability-weighted auxiliary loss on position channels), ABot-World-0 (soft flags → sample weights).

**2. The objective determines the curation bill.** Being-H0.7's latent-future alignment needs only temporal coherence (200K h); video-prediction needs generable pixels; pseudo-action labelling needs accurate hand tracking and is the most expensive. DreamZero states it directly: world modeling *"enables effective learning from diverse demonstrations."*

**3. Alignment is a precondition for scaling, not a nicety.** Qwen-RobotManip's ablation: with unified EEF representation, validation MSE falls log-linearly with data; without it, **no scaling behaviour at all.** π₀.₇'s mirror finding: with metadata conditioning, performance improves as average data quality *falls*; without it, performance degrades.

**4. A small aligned anchor unlocks a large noisy corpus.** Independently reached by EgoScale (829 h EgoDex inside 20,854 h), EgoVerse (2 h domain-aligned unlocks 8 h diverse), Galaxea (single-embodiment bridge stage), Ψ₀ (post-training on 30 h robot data).

**5. Filtering is distribution shift.** Only Cosmos 3 measures it — retention by capability category, finding referring-expression grounding pruned most aggressively as thresholds rise.

**6. Diversity beats density past a threshold.** EgoVerse (scene coverage > within-scene density), DreamZero (hour-matched diversity > repetition), GEN-0 (*"data quality and diversity matters more than sheer volume"*), AXIS (volume-matched baseline beaten by 37.3%).

**7. Success-only corpora cannot teach recovery.** Kairos names the missing categories precisely — near-boundary failures, recovery events, **near-boundary successes**, contact transitions, safety anomalies, long-horizon dependencies.

**8. Disclosure is inversely correlated with neither capability nor geography-neutral.** The `A+` tier is Alibaba (×3), NVIDIA (×2), ACE, Kairos. The `C`/`D` tier is Google DeepMind, Figure, Skild, 1X, AGIBOT. The most reproducible pipelines in 2026 come from Chinese industry labs and open academic groups.

## Open disagreement worth tracking

**Volume vs. purity of human video.** Ψ₀: ~800 h of clean EgoDex beats baselines pretrained on >10× more data, by >40% absolute. DYNA-2: 1,000,000 h of unprocessed video, with *no* visual or embodiment-specific gap reduction, yields the first cross-embodiment transfer scaling law. EgoScale sits between, with the only fitted curve (R²=0.9983) and explicit tolerance for noisy labels. These cannot all be the whole story, and the field has not yet run the experiment that separates them.
