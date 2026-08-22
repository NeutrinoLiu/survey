# PRISM — Precision and Contact-Rich Real-world Industrial Skill Dataset with Multimodal Sensing

**arXiv:2608.17962** · Peking University + Delta Intelligence + PKU-Wuhan Institute for AI + Hubei Humanoid Robot Innovation Center + CAICT (T. Yu, J. Wu, H. Wang, R. Chen, C. Liu, C. Sun, H. Liu) · Aug 2026 · [site](https://tengbo-yu.github.io/PRISM/)

**One line.** Argues that robot-learning datasets are **structurally mismatched to industry** — they are short-horizon, low-contact and household — and collects 45 hours of actual industrial assembly with force/torque and tactile.

## 1. The mismatch

*"Most existing datasets emphasize short-horizon, low-contact tasks such as pick-and-place, and therefore do not capture the precision control, force/torque or tactile regulation, and multimodal feedback required for industrial assembly."*

Why vision alone fails in this regime, stated concretely: *"minute misalignments, frictional sticking, compliance, and insertion jamming can be visually ambiguous but are immediately expressed through force/torque or tactile signals."*

And the gap beyond scale: *"Existing resources rarely combine multi-embodiment robot platforms, synchronized multimodal sensing, industrial contact-rich procedures, and diverse teleoperation interfaces within one unified dataset."*

## 2. Dataset

| | |
|---|---|
| Tasks | **25+** manipulation |
| Trajectories | **5,000+** teleoperated |
| Duration | **45+ hours** |
| Modalities | multi-view **RGB-D**, **six-axis force/torque**, **tactile** (subset of episodes), robot state |

Tasks are genuinely industrial rather than household proxies: **electronic component plug/unplug, conveyor-based sorting, installing bearings, automotive material sorting**, spanning *"diverse mechanical constraints representative of real industrial processes."*

Multi-embodiment, multiple teleoperation interfaces — so policies must transfer *"across robot bodies, sensing modalities, operators, and contact conditions in production-like settings."*

## 3. What it adds that the others don't

**Domain realism.** Every other dataset in this survey collects contact-rich tasks *chosen to be contact-rich* — peg insertion, wiping, peeling — in a lab. PRISM collects tasks that are contact-rich because a factory requires them, with the tolerances, cycle structure and mechanical constraints that come with that. That matters for two reasons: the tolerance distribution is genuinely different (tight-fit assembly rather than a 3 mm-clearance peg), and long-horizon procedures compound in ways single-skill benchmarks do not expose.

It also inherits the honest caveat that **tactile is present only on a subset of episodes** — force/torque is the primary contact modality — which reflects the practical reality that gel sensors are hard to run for 45 hours of industrial contact. Read alongside [[vitatouch]], the other industrial-domain entry here, which uses touch for inspection rather than assembly.
