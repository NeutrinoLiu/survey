# HTT — Heterogeneous Tactile Transformer

**arXiv:2606.29948** · NUS + CMU + Smart Systems Institute NUS (Bi, Q. Wang, Reddy, K. Lin, Khajikhanov, R. Gao, Soh) · Jun 2026

**One line.** Bridges the **optical vs. array** divide — the two tactile sensor families with genuinely different output structures — by collecting **1.6M time-synchronised paired frames** and training a shared trunk across them.

## 1. What "tactile" means here — two families, deliberately

| Family | Examples | Output | Strength | Weakness |
|---|---|---|---|---|
| **Optical** | GelSight Mini, 9DTact, DIGIT | images of elastomer deformation | rich spatial/geometric detail | limited by camera frame rate; sensor-to-sensor variation |
| **Array-based** | Xela uSkin, TAC-02, PapillArray | time series over sensing elements | high-rate, direct force/pressure | low spatial resolution |

The problem is stated structurally: *"their raw outputs have very different structures: images in one case and time series over sensing elements in the other."* Prior representation-learning work is *"largely focused on optical tactile sensors"* — so those models cannot ingest array signals at all and are poorly suited to tasks depending on high-rate force or slip cues.

## 2. Data curriculum — the HPT dataset

**1,590,000 synchronized paired frames** across four sensors, collected with a **UMI** device:

| Sensor | Frames | Share |
|---|---|---|
| TAC-02 (array) | 746k | 47% |
| Xela (array) | 604k | 38% |
| GelSight Mini (optical) | 133k | 8% |
| 9DTact (optical) | 103k | 6% |

Motions cover **press, twist, and slide**. The claim — *"to our knowledge, there is still no large-scale dataset of time- and space-synchronized optical and array-based tactile observations"* — appears correct for 2026; every other paired dataset in this survey ([[tacverse]], TacQuad) pairs optical with optical.

## 3. Model

**Sensor-specific encoders + shared transformer trunk**, with two pretraining objectives:

1. **Per-modality masked reconstruction** (MAE-style) — each sensor's data is patchified, encoded, passed through the shared trunk, and reconstructed by a sensor-specific decoder.
2. **Bidirectional cross-modal alignment** — a cross-modal predictor takes an *unmasked* source embedding `z_i` and predicts the *masked* target embedding `sg[z_j]` from the paired sensor, with **stop-gradient** on the target.

The design intent: *"preserve sensor-specific structure while aligning representations across sensors"* — private encoders for structure, shared trunk for alignment. Structurally the same bet as [[ftp-1]]'s type-specific encoders into a shared token space and [[n0-twam]]'s per-modality experts under shared attention.

## 4. Results

**Real-world manipulation** — and the setup is the aggressive part: a **Sharpa hand on a Franka arm with no external camera at all**. The policy has only proprioception and fingertip tactile. Crucially, **the Sharpa fingertip sensors were never seen during HTT pretraining** — the 9DTact encoder is applied zero-shot.

Two tasks: **toy screw** (repeatedly grip and rotate until tightened; >600° counts as success) and **grasp tofu** (lift soft tofu without crushing or dropping). ACT policy, 20 demos for screw, 50 for tofu.

| Observation | Toy screw | Grasp tofu |
|---|---|---|
| qpos only (22-D joints) | 5% | 5% |
| + 6-D wrench per fingertip (×5) | 50% | 35% |
| **+ HTT embeddings** | **95%** | **55%** |

The qpos-only collapse confirms the tasks are genuinely unsolvable without touch in a camera-free setup — a cleaner control than most tactile papers manage.

**The screw result has a mechanistic explanation worth keeping.** In **19/20** rollouts HTT maintains grip across two rotation cycles and fully tightens, while the wrench policy *"loses contact during the second cycle and stalls."* The authors' interpretation: cyclic regrip depends on **spatial contact features — where on the finger pad contact is made and how it shifts during regrip — which a 6-D wrench vector cannot represent.** That is the clearest task-level demonstration in this survey of *why* spatially resolved tactile beats aggregated force, and it directly complements [[at-vla]]'s opposite finding (force 6D > images) — the two disagree because the tasks differ, not because one is wrong.

**Tofu failure analysis** is refreshingly granular: slip during lift dominates (**12/20 wrench, 8/20 HTT** — a one-third reduction). Wrench **crushes** the tofu in 1/20; HTT **never crushes**, but in one rollout fails to trigger the lift because the embedding *"does not register sufficient fingertip contact."* Different representations produce different failure distributions, and HTT's are the safer kind.

**ManiFeel simulation** (success rate, 3 seeds × 50 rollouts):

| Method | Peg Insertion | Bulb Installation |
|---|---|---|
| tacRGB | 0.21 ± 0.02 | 0.72 ± 0.04 |
| T3 | 0.23 ± 0.02 | 0.73 ± 0.06 |
| SITR | 0.35 ± 0.01 | 0.77 ± 0.04 |
| **HTT (RGB)** | **0.44 ± 0.04** | 0.77 ± 0.02 |
| **HTT (FF)** | **0.48 ± 0.12** | 0.76 ± 0.02 |

Peg insertion doubles over tacRGB; bulb installation **converges near 0.77 for every method** — a saturated task where the representation no longer matters, worth noting since papers rarely flag their own ceilings.

## 5. Stated limitations, all pointed

- Only optical and array families; magnetic and fluid-based sensing untouched.
- Pairing is **exclusively cross-family** — the effect of pairing two optical or two array sensors is unexplored.
- **HPT pairs sensors in time and contact but not in geometric space**: which spatial patches of an optical sensor correspond to which taxels of an array sensor is not modelled, leaving finer-grained cross-modal supervision on the table.

## 6. What it adds that the others don't

The **optical↔array pairing**. Every other cross-sensor effort here ([[tacverse]], [[anytouch2]], [[ftp-1]], [[tactx]], [[unitac]]) works within or across optical gels; HTT is the one that unifies the two families whose raw formats are genuinely incommensurable, and demonstrates zero-shot transfer to a fifth, unseen sensor on a camera-free robot. The screw-task analysis — spatial contact layout is what a wrench vector cannot carry — is the sharpest available argument for why that unification is worth the effort.
