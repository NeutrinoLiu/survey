# Splash ("Wake up for Touch!") — Mask-isolated Tactile Alignment Learning in MLLMs

**arXiv:2607.00302** · Ewha Womans University (Y. Park, M. Kim, S. Moon, J. Lee) · Jul 2026 · [site](http://mmai.ewha.ac.kr/splash/)

**One line.** Identifies a **zero-sum trade-off** in small MLLMs — adding touch destroys vision-language ability — and resolves it by confining all tactile learning to the model's **dormant (least important) parameters** while freezing the critical ones.

## 1. The problem, demonstrated

The failure case is vivid. A Qwen2.5-VL-3B asked to describe an image scores 9/10. The same model after tactile training on TVL, without mask-guided alignment, hallucinates *"a black, flexible rubbery tool, possibly a rubber band"* stretched over a can — 5/10. **A small tactile training set forgets the visual sense in the base MLLM.**

The framing: *"the limited parameter budget of compact models forces a choice between acquiring the new sensory modality and preserving the established vision-language reasoning."*

Why small models specifically: MLLMs with 7B+ backbones *"limit their practicality for resource-constrained edge robots"*, so ≤3B models are the ones actually wanted — and they are the ones where the trade-off bites.

The paper also motivates MLLMs over contrastive VTL representations: shared visuo-tactile-language spaces are *"effective for discriminative tasks such as retrieval and classification"* but *"lack the capacity for the higher-level semantic reasoning and instruction following that complex embodied decision-making demands"*, and *"generalize poorly to novel objects with visually ambiguous textures."*

## 2. Method

1. **Estimate parameter importance** with a **VL-relative metric** computed from weight and activation statistics.
2. **Partition** the parameter space into a **critical** subspace (frozen — a stable vision-language anchor) and a **dormant** subspace (less important).
3. **Restrict gradient updates to the dormant subspace**, trained jointly with a small tactile front-end.

Three practical properties follow:
- **No added inference cost** in the LLM — no adapters, no external modules, original model scale preserved
- **Single-stage training** — unlike TVL and similar, which need multi-stage alignment then fine-tuning
- **Front-end agnostic** — any tactile encoder can be substituted

## 3. Results

State-of-the-art on **SSVTP, TVL and TacQuad**, with **Splash-3B outperforming previous 7B-scale methods**.

The paper also contributes *"the first comprehensive performance analysis of preservation in VTL alignment across complex general-purpose VL benchmarks"* — i.e. it measures what tactile training *costs* on the original tasks, which the tactile-language literature had not been reporting. The result: alignment with Splash *"does not compromise the broad-spectrum intelligence of MLLM backbones."*

**Qualitative comparison on TacQuad** (LLM-judge scores), on a plastic bowl handle:

| Model | Prediction | Score |
|---|---|---|
| UniTouch | "1. Round 2. Smooth 3. Shiny" | 3/10 |
| TVL-LLaMA | "smooth, flat, rigid" | 3/10 |
| **Splash-1B** | "smooth, hard, cool, glossy, flat" | **8/10** |
| **Splash-3B** | "smooth, reflective, hard, cool, glossy" | **8/10** |

The diagnosis of the baselines is the same pathology [[touch-r1]] targets: *"baselines rely more heavily on visual appearance cues when tactile grounding is weak"*, generating *"visually plausible but tactually inconsistent attributes (e.g. smooth or reflective for coarse surfaces)."* Note that both baselines omit **temperature** ("cool"), which is a purely tactile property with no visual correlate.

## 4. Stated limitations

- **The critical subspace is static** after an offline splitting step. But *"in practical robotic manipulation scenarios, activation patterns may dynamically change depending on interaction state, contact condition, or task progression"*, so a fixed dormant subspace may not capture temporally varying parameter utilisation over long-horizon reasoning. Dynamically adaptive masking is proposed as future work.
- **Single DIGIT sensor**; cross-sensor generalisation untested.

## 5. What it adds that the others don't

**Catastrophic forgetting as the central object of study**, and a parameter-space rather than architecture-space solution. Several papers here worry about disturbing pretrained priors — [[at-vla]] gates, [[taco]] stop-gradients, [[vitar]] freezes entirely, [[modsensorystream]] pre-aligns — but Splash is the only one that *localises* the conflict to specific weights and updates only the ones the base capability does not use. Reporting the preservation cost on general VL benchmarks is a reporting practice the tactile-language cluster should adopt wholesale.
