# Gemini Robotics 2 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Gemini Robotics 2 · Gemini Robotics ER 2 · Gemini Robotics On-Device 2 |
| **Org** | Google DeepMind |
| **Date** | 2026-07-30 |
| **Artifacts** | `page.html` (DeepMind blog), `modelcard.html` (Gemini Robotics ER 2 model card) |
| **Disclosure level** | ⚠️ **D — the least data disclosure of any work in this survey.** The model card's entire "Model Data" section defers to the Gemini 3.5 Flash card. No robotics corpus size, sources, mixture, or processing is described anywhere. |
| **Stance** | Not stated. Data is treated as undisclosed infrastructure. |

## 1. What is actually disclosed about data

The complete published content of the **Training Dataset** field:

> *"Gemini Robotics ER 2 was trained on **Gemini 3.5 training datasets and additional datasets representing various embodied reasoning tasks**. For more information about the training data, see the Gemini 3.5 Flash model card."*

And the complete content of **Training Data Processing**:

> *"For more information about the training data processing, see the Gemini 3.5 Flash model card."*

That is the entirety of it. Architecture: ER 2 is based on **Gemini 3.5 Flash**; trained on TPUs with JAX and ML Pathways.

This is worth recording precisely because Gemini Robotics is among the most capable systems in the field. **Capability and data disclosure are uncorrelated**, and any survey of curation practice has to note that the frontier includes participants who publish nothing about it.

## 2. What can be inferred about the data requirement

The blog describes capabilities whose data implications are legible even when the corpus is not.

**Three-model stack:**
| Model | Role | Data implication |
|---|---|---|
| **Gemini Robotics 2** | VLA; vision+language → motor control; *"controlling full humanoids, from feet to fingertips, and other bi-arm robots"* | Requires **whole-body** trajectories including locomotion, not arm-only manipulation data |
| **Gemini Robotics ER 2** | Embodied-reasoning VLM; human communication, physical-world understanding, **multi-step plans lasting several minutes**, multi-robot teaming | Requires long-horizon planning supervision and multi-agent episodes |
| **Gemini Robotics On-Device 2** | Efficient local VLA | — |

**Cross-embodiment from one checkpoint.** The same checkpoint controls **Apptronik Apollo 2 with SharpaWave hands (22-DoF, five-fingered)**, **Apollo 2 with Inspire hands**, and **Franka Duo with Robotiq gripper** — implying an action representation and a corpus spanning radically different end-effectors, in the same family of problem that Qwen-RobotManip, ABot-M0, and ACE-Ego-0 solve with explicit harmonization schemes.

**Adaptation cost as an implicit data statement:** *"fast adaptation to completely new robot embodiments with **a few hours of data**"* — the same order as π₀.₇ and GEN-1 (~1 h), and a strong claim about the value of whatever pretraining corpus underlies it.

## 3. Reported results (for calibration, not method)

Apollo 2:
| Task setting | Success |
|---|---|
| Picking from a table | 68.4% |
| Picking from the floor | 45.7% |
| Picking from a shelf | 76.3% |
| General pick-and-place | 74.2% |
| Tool kitting | 78.9% |
| Precise insertion | 89.6% |

DeepMind's own honest caveat: *"While Gemini Robotics 2 achieves a medium to high success rate for whole-body and gripper-based dexterous tasks, **the multi-finger dexterous manipulation remains challenging**."*

## 4. Safety data — the one area with substantive discussion
ER 2 is described as their safest robotics model to date, with embodied reasoning that *"can better detect when humans are nearby, trigger safety tool calls, and bring the robot to a safe stop if someone approaches too closely."* This implies dedicated human-proximity and safety-intervention training data, but no collection or curation detail is given.

## 5. Why it is included here
Three reasons:
1. It is a **frontier reference point** — any claim that a curation practice is necessary must contend with the fact that the strongest published whole-body results come from a system whose data practice is unknown.
2. It **bounds what the field can conclude.** Comparative statements about corpus size or cleaning strategy cannot include the closed frontier.
3. The **capability profile is itself evidence** about data requirements: whole-body control, multi-minute planning, multi-robot coordination, and three end-effector classes from one checkpoint each imply corpus properties that open datasets currently lack.

## 6. Transferable takeaways
1. ⚠️ **Non-disclosure is the norm at the closed frontier.** Google, Figure, Skild, and 1X publish strategy or results but not pipelines; the reproducible detail in this survey comes overwhelmingly from Chinese industry labs (Alibaba, Ant, Tencent, AGIBOT) and academia.
2. **Deferring a robotics model card's data section to a general LLM card** obscures exactly the robot-specific curation questions a reader needs answered.
3. **Adaptation cost is a readable proxy** for pretraining corpus quality when the corpus itself is undisclosed — "a few hours to a new embodiment" is a substantive claim even without a data table.
