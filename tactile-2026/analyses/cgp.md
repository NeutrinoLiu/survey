# CGP — Contact-Grounded Policy: Dexterous Visuotactile Policy with Generative Contact Grounding

**arXiv:2603.05687** (v3, May 2026) · Purdue + **Meta Reality Labs Research** + UW-Madison (Z. Xu, Y. Wang, Abbatematteo, Preechayasomboon, Chan, Colonnese, Memar) · Mar 2026 · [site](https://contact-grounded-policy.github.io/)

**One line.** Closes the gap most tactile policies leave open — **how a predicted contact becomes a controller command** — with a learned *contact-consistency mapping* from (predicted state, predicted tactile) to compliance-controller targets.

## 1. The gap

*"Most [tactile-informed policies] use tactile signals as additional observations rather than modeling contact state or **how their action outputs interact with low-level controller dynamics**."*

Under a compliance controller, the commanded target is *not* the achieved state — the controller yields to contact forces. So a policy that outputs desired states without accounting for that yielding cannot realise an intended contact.

## 2. Model — two components

1. **Conditional diffusion model** forecasting coupled future trajectories of **actual robot state and tactile feedback** in a compressed latent space.
2. **Learned contact-consistency mapping** converting the predicted (state, tactile) pair into **executable targets for a compliance controller**, *"enabling it to realize the intended contacts."*

Two ablated design choices, both validated:
- Conditioning the mapping on **proprioceptive state** as well as tactile beats a tactile-only variant.
- Predicting **residual corrections** beats predicting absolute hand configurations.

## 3. Hardware

Physical **four-finger Allegro V5** with **Digit360** fingertips, and a simulated five-finger **Tesollo DG-5F** with **dense whole-hand tactile arrays** — covering both vision-based and dense-array tactile. Demonstrations collected by teleoperation in **both simulation and on the physical robot**.

## 4. Results

Outperforms visuomotor and visuotactile diffusion-policy baselines across in-hand manipulation, delicate grasping and tool use — fragile egg grasping, in-hand box flipping, dish wiping, jar opening.

**The target-vs-actual analysis is the most illuminating figure in the policy cluster.** Overlaying commanded and achieved joint states through a rollout:

> *"Before contact, the small target-actual mismatch primarily reflects the steady-state tracking error of the low-level PD controller (e.g. due to gravity and friction). **During contact, the gap grows as the joint-space compliance controller yields to contact forces; in this phase, the gap reflects the physical interaction.** When a finger loses contact, the contact forces disappear and the gap quickly shrinks."*

The target–actual gap *is* a contact signal. That reframes compliance-controller tracking error from a nuisance into information, and explains why a policy must model it rather than assume its commands are executed.

**Robustness:** injecting Gaussian noise into tactile inputs at varying fractions of the dataset-level tactile standard deviation leaves the mapping's prediction error stable across a wide range — *"errors in tactile prediction do not catastrophically propagate to the mapped robot state."* Given that predicted tactile is imperfect everywhere in this survey, that decoupling matters.

## 5. What it adds that the others don't

**Taking the low-level controller seriously.** Almost every tactile policy here outputs positions and assumes execution; CGP models the compliance controller's yielding explicitly and learns the inverse mapping needed to command an intended contact through it. Together with [[forcevla2]]'s hybrid force–position action space, it is one of only two works in the survey where the *action representation* is adapted to contact rather than only the observation representation.
