# LAP (Language-Action Pre-training) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | LAP: Language-Action Pre-Training Enables Zero-shot Cross-Embodiment Transfer |
| **Org** | Princeton University + Physical Intelligence |
| **Date** | 2026-02 (arXiv 2602.10556v2) |
| **Artifact** | `paper.pdf`, `paper.html`; lap-vla.github.io; github.com/lihzha/lap |
| **Disclosure level** | **A — full paper, code released** |
| **Stance** | **Eliminate the action-representation problem entirely: express actions as natural language.** |

## 1. Why it belongs here

Most of this survey is about the cost of harmonizing heterogeneous action data — Qwen-RobotManip's canonical state-action vector, ABot-M0's delta-EEF + axis-angle conversion, ACE-Ego-0's three-axis alignment, GR00T's latent-action codebook, Qwen-VLA's embodiment prompts. LAP proposes that this entire layer is avoidable:

> *"we introduce **Language-Action Pre-training (LAP)**, a simple recipe that **represents low-level robot actions directly in natural language**, aligning action supervision with the pre-trained vision–language model's input–output distribution. LAP requires **no learned tokenizer, no costly annotation, and no embodiment-specific architectural design**."*

The three "no"s are each a **pipeline stage that other works must build and that LAP deletes**:
| Deleted stage | Who else needs it |
|---|---|
| Learned action tokenizer | Ψ₀ (retrained FAST tokenizer on 500K actions), GR00T (latent-action codebook), π₀-FAST |
| Costly annotation | Everyone deriving pseudo-actions from video |
| Embodiment-specific architecture | ACE-Ego-0 (morphology tokens), GR00T (per-embodiment encoders/decoders), ABot-M0 (pad-to-dual), LingBot (per-domain heads) |

## 2. The problem it targets

> *"Despite large-scale multi-embodiment pre-training, existing VLAs **remain tightly coupled to their training embodiments and typically require costly fine-tuning**."*

This is the same finding Galaxea reports from the other direction — that cross-embodiment pretraining degrades when the embodiment gap is large — and it identifies the coupling as a *representation* problem rather than a data-volume problem.

## 3. The mechanism and its data consequence

By expressing actions in the VLM's native output distribution (text), the action data becomes **the same kind of object as the language data**. The practical consequences for a data pipeline:

- **No numeric normalization needed.** No per-embodiment quantile statistics, no `[q01, q99] → [-1,1]` maps, no rotation-representation conversions — the whole class of defects that Qwen-RobotManip's five-stage filter exists to catch (sign conventions, frame mismatches, extreme values) is bypassed or absorbed.
- **Action prediction and VQA unify.** *"unifying action prediction and VQA in a **shared language-action format** that yields additional gains through co-training."* Where Qwen-RobotManip curates a separate 28M-sample VL stream and GR-3 builds a separate VL corpus to prevent semantic erosion, LAP makes the two objectives the *same* objective — there is nothing to balance.
- **Adding an embodiment costs nothing structurally.** New robots do not require new encoders, new action dimensions, or new padding schemes.

## 4. Result
**LAP-3B**: *"to the best of our knowledge is the **first VLA to achieve substantial zero-shot transfer to previously unseen robot embodiments without any embodiment-specific fine-tuning**. Across multiple novel robots and manipulation tasks, LAP-3B attains **over 50% average zero-shot success, delivering roughly a 2× improvement** over the strongest prior VLAs."*

Also reports *"efficient adaptation and **favorable scaling**."*

## 5. Trade-offs and limits
- Natural-language action encoding is **lower precision** than continuous flow-matching output; the 50% zero-shot figure is strong for transfer but below in-distribution specialists.
- Text tokens are a costly channel for high-frequency control — a throughput constraint that continuous action experts (π₀.₇'s 860M DiT, GR00T's 120 Hz System 1) exist to solve.
- No corpus description, filtering pipeline, or mixture table is published; the contribution is representational.

## 6. Position in the survey
LAP is the strongest available argument that **much of the 2026 curation effort is a consequence of a representation choice rather than an inherent property of robot data.** If actions live in the VLM's native output space, the alignment problem that Qwen-RobotManip identifies as the precondition for log-linear scaling largely dissolves. Whether that holds at the precision and frequency real dexterous manipulation requires is the open question.

## 7. Transferable takeaways
1. **Ask whether an alignment pipeline is solving a real problem or a self-inflicted one.** Many harmonization stages exist only because actions were encoded outside the backbone's native distribution.
2. **Aligning supervision with the pretrained model's input–output distribution removes the need for a tokenizer** and for embodiment-specific architecture.
3. **Unifying action and VQA into one format eliminates the co-training balance problem** that most VLA pipelines must tune.
4. **Zero-shot transfer to unseen embodiments is the right test** of whether cross-embodiment data was actually harmonized or merely pooled.
