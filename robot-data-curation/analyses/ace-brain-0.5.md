# ACE-Brain-0.5 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | ACE-Brain-0.5: A Unified Embodied Foundational Model for Physical Agentic AI |
| **Org** | ACE-Brain Team |
| **Date** | 2026-07 |
| **Artifact** | arXiv 2607.04426 (`paper.pdf`, `paper.html`); github.com/ACE-Brain-Team/ACE-Brain-0.5; HF `ACE-Brain/ACE-Brain-0-8B` |
| **Lineage** | ACE-Brain-0 (2603.03198, *Spatial Intelligence as a Shared Scaffold for Universal Embodiments*) → **ACE-Brain-0.5** (latest) · sibling data work: **[ACE-Ego-0](../ace-ego-0/dataprocess.md)** |
| **Disclosure level** | **A — full report with §5 Data and Benchmark** |
| **Model** | Single **8B** backbone spanning five coupled functions |
| **Stance** | **Cross-interface interference is a data-mixing problem, and it is solved by *not* mixing.** |

## 1. The five coupled functions and their incompatible output interfaces

ACE-Brain-0.5 organizes robot intelligence into **spatial perception · decision making · embodied interaction · self-monitoring · self-improvement**, extending ACE-Brain-0 *"from an understanding-centric spatial model into a closed-loop embodied model that can perceive the physical world, plan under goals, act through robot bodies, monitor execution progress, and improve from accumulated experience."*

The problem this creates is a **data-mixture** problem, and the paper names it precisely:

> *"[Grounding produces] region or point outputs; **Navigation** requires sequential action prediction under egocentric observations; **Manipulation** requires continuous action chunks; **Progress estimation** requires temporally grounded evaluation of execution states. **Directly mixing all data sources in a single supervised fine-tuning (SFT) stage leads to cross-interface interference**: the model may **retain the semantic knowledge for a task but fail to follow the correct output convention for that interface**, or specialize one interface at the cost of others."*

This is a distinct failure mode from everything else catalogued in this survey. It is not corrupted data, not misaligned coordinate frames, not mixed quality — it is **semantically correct data whose output *conventions* conflict**. Naive mixing produces a model that knows the answer but emits it in the wrong format.

## 2. The remedy — train separately, merge, then reactivate

Instead of solving mixture weights, ACE-Brain-0.5 avoids the mixture:

```
train per-interface specialists  →  MODEL MERGE (θ_merge)  →  lightweight REACTIVATE stage
                                                                θ_0.5 = SFT(θ_merge, D_mix)
```

> *"We therefore apply a lightweight **Reactivate** stage: θ_merge is fine-tuned on a **compact mixed SFT dataset D_mix for a small number of updates**."*

Justified from the model-merging literature: *"a **brief warm-up on mixed data reliably restores or even improves upon individual task performance**."*

**The curation consequence is notable:** the expensive, carefully-balanced mixture other works agonize over (ABot-M0's Gini analysis, Qwen-VLA's proportion table, GR00T's pyramid sampling) is replaced by **per-interface corpora plus a small mixed set used only to re-couple the merged weights**. `D_mix` needs to be *representative*, not *balanced* — a much weaker and cheaper requirement.

## 3. Recovery-data construction with diagnostic metadata

For self-monitoring and recovery, supervision pairs a predicted action `â_t` with an **oracle recovery action** `a*_t`, plus:

> *"ρ_t records **offline deviation metadata, such as progress state, distance-to-goal change, path-alignment status, and accumulated local error**. This metadata is **used only for diagnosis, filtering, and supervision construction, and is not provided to the policy during evaluation**."*

Two important properties:
1. **Metadata as a construction and filtering tool, not a model input.** This is the deliberate opposite of π₀.₇, which feeds quality metadata *to* the model as conditioning. ACE-Brain uses it upstream to *build* and *filter* the supervision, then withholds it — avoiding any train/test information leak.
2. **Four concrete deviation signals** (progress state, distance-to-goal delta, path alignment, accumulated local error) that are cheap to compute offline and give a per-timestep handle on "how wrong was this?" — a usable recipe for anyone constructing recovery data.

## 4. Results
ACE-Brain-0.5-VLA reaches **82.3%** average success on SimplerEnv-Bridge (vs GTA-VLA 81.2%, X-VLA 76.0%, Qwen-VLA-Instruct 73.7%, Uni-VLA 69.8%). Notably, *"given that there is sufficient training data on the Bridge dataset, we **did not use the pre-training weights of ACE-Brain-0.5 on manipulation**"* — an honest disclosure that the manipulation result does not depend on their own pretraining.

Also carries **affordance annotations** among its supervision types.

## 5. Relationship to ACE-Ego-0
The two ACE works divide the problem: **ACE-Ego-0** builds the egocentric video-to-action data pipeline (five stages, published thresholds, reliability-aware supervision), while **ACE-Brain-0.5** addresses how to combine heterogeneous *task interfaces* into one backbone. Read together they cover both halves of the mixture problem — heterogeneous *sources* and heterogeneous *output conventions*.

## 6. What they do not do
- No corpus sizes, source tables, or mixture proportions published.
- No filtering thresholds or rejection rates.
- The merge-and-reactivate recipe's sensitivity to `D_mix` composition is not ablated in detail.

## 7. Transferable takeaways
1. **Cross-interface interference is a real, distinct failure mode.** Data can be individually clean and jointly harmful when output conventions conflict.
2. **Train specialists, merge, then reactivate** — this sidesteps mixture-weight tuning entirely and needs only a small representative mixed set.
3. **Compute deviation metadata offline** (progress, distance-to-goal change, path alignment, accumulated error) to construct and filter recovery supervision.
4. **Decide deliberately whether metadata is an input or a tool.** π₀.₇ conditions on it; ACE-Brain withholds it to avoid leakage. Both are defensible, but the choice must be explicit.
