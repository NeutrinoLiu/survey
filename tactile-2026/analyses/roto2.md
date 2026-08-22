# roto 2.0 — The Robot Tactile Olympiad

**arXiv:2605.21429** · University of Edinburgh + NUS (Miller, Reddy, Deshmukh, McInroe, Abel, Mac Aodha, Vijayakumar) · May 2026 · [site](https://elle-miller.github.io/roto/)

**One line.** A GPU-parallelised RL benchmark for *blind* dexterous manipulation across four hands, built on the argument that the field over-fitted to in-hand reorientation and that binary contact plus proprioception is a stronger baseline than anyone assumed.

## 1. What "tactile" means here

Deliberately minimal: **binary contact flags**, one per task-relevant robot link. No forces, no shear, no images, no taxel arrays — a boolean per link, stacked over a history of **k = 4** timesteps.

That impoverishment is the point. The authors want to know how far a policy gets on the least informative tactile signal available, and they explicitly name "beyond sparse binary contacts to richer forms of tactile information" as the top open direction. Sensor simulation fidelity is thus sidestepped entirely, which is also why sim-to-real is unresolved.

**Blind** means proprioception + tactile only: no object state, no pose estimator, no teacher-student distillation. State-based agents (privileged) additionally get object position(s) and linear velocity.

Four morphologies, 16–24 DOF: **Shadow Dexterous Hand** (24), **Shadow Lite** (16), **Allegro** (16), **ORCA** (17).

## 2. Data curriculum

None — this is on-policy RL, not imitation. The "curriculum" is compute: **8,092 parallel environments** for training, 100 held out for continuous evaluation, physics at 240 Hz and control at 60 Hz, built on Isaac Lab.

What substitutes for a data pipeline is **tuning discipline**: for every (robot × task × observation setting) combination, a hyperparameter sweep over seven PPO parameters, 40 trials with 8 warm-up runs. The stated motivation is to let researchers "prioritise fundamental algorithmic challenges over tedious RL tuning" — the released artifact is as much the tuned baselines as the environments.

## 3. Model

Customised **PPO** (SKRL implementation) with observation stacking, self-supervision hooks, and separated evaluation environments. Joint-position control.

## 4. How tactile enters the model

Concatenated into the observation vector alongside joint positions, joint velocities, joint command error, and last action, stacked over k = 4 timesteps. No tactile encoder at all. The prior work this builds on (Miller et al., NeurIPS 2025) shows that adding a **self-supervised forward-dynamics objective** for representation learning is what unlocks the top numbers — the authors identify "efficient feature extraction" as the core bottleneck of tactile RL.

## 5. Experiment setup

Two tasks, both 10 s / 600 timesteps:

- **Bounce** — bounce a ball as many times as possible; a bounce counts as a contact event after ≥5 timesteps (~83 ms) without contact. Reward r = 10 per bounce; theoretical max return 1,000 = 100 bounces.
- **Baoding** — rotate two 55 g balls around each other in-hand. Ball diameter is scaled per hand (1.5" Shadow/ORCA, 2" Allegro, 1.2" Shadow Lite). Reward = r_dist1 + r_dist2 + r_rotation, with two static targets that swap once both ball centres are within 1.0 cm, granting a +10 bonus.

5 seeds per configuration.

## 6. Does tactile actually help?

The headline is a **capability** result rather than an ablation: blind agents reach **13 Baoding rotations in 10 s**, against a prior state of the art of 3 rotations (Robot Synesthesia). With the self-supervised forward-dynamics addition, up to **25 rotations**, versus **35** for privileged state-based agents. Blind policies get within striking distance of privileged ones without distillation or pose estimation — which is the paper's real claim: *haptic foundations are a prerequisite to adding vision, not an afterthought to it.*

Task-dependence is stark and honestly reported:

- **Bounce** — blind agents are highly sample-efficient, approaching 80 bounces by 200M steps; performance trends stay consistent across morphologies except Shadow Lite. Blind ≈ privileged.
- **Baoding** — a large gap opens. Blind agents show "much lower performance with high variance": a top Shadow Hand seed reaches 13 rotations while **other seeds fail to converge entirely**. Multi-object manipulation from sparse contact remains open.

Morphology matters less than expected — despite 16 vs 24 DOF and very different kinematics, the full-hand morphologies behave similarly; only the reduced Shadow Lite breaks the trend. Hands adopt hardware-specific strategies (ORCA settles on an "outstretched, starfish-like" pose for bouncing).

**Honest limits, stated plainly.** These are simulated policies representing a "performance ceiling that exceeds current real-world hardware limits"; sim-to-real transferability is under active investigation, not demonstrated. And the tactile signal is binary contact, so nothing here speaks to what richer sensing would buy — the authors say so and list it as the first research direction.

## 7. What it adds that the others don't

The only 2026 benchmark that isolates **tactile-driven RL without vision, without privileged state, and without distillation**, across multiple hand morphologies — and the only one whose result is that the tactile-proprioceptive loop alone is far more capable than the literature's reorientation focus suggested. Where [[taco-bench]] asks *which sensor helps an imitation policy*, roto asks *how far pure touch can go with enough compute*, and answers: further than expected on single-object tasks, not yet on multi-object ones.
