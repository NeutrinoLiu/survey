# GEN-0 / GEN-1 (Generalist AI) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **GEN-0**: Embodied Foundation Models That Scale with Physical Interaction (2025-11-04) · **GEN-1**: Scaling Embodied Foundation Models to Mastery (2026-04) |
| **Org** | Generalist AI |
| **Artifacts** | `gen0.html`/`gen0.txt`, `page.html`/`page.txt` (generalistai.com blog) |
| **Disclosure level** | **B — detailed company research blog.** Real ablation tables, a fitted power law, and named metrics; but no paper, no corpus release, no data-partner identities, and no cleaning-rule specifics. |
| **Corpus** | **270,000 h** (GEN-0, Nov 2025) → **>500,000 h** (GEN-1, Apr 2026); growing **>10,000 h/week** |
| **Stance** | **Data as an operations business.** The pipeline is a global collection network plus continuous A/B testing of *mixtures*, not a filtering cascade. |

## 1. Two headline claims about data

**(a) The base model contains no robot data.**
> *"the base foundation model is trained without any robot data—it instead uses data from low-cost wearable devices on humans doing millions of activities, and provides an existence proof that this pretraining can lead to high levels of mastery without requiring large teleoperation or simulation datasets."*

GEN-1 adapts to a new task with **~1 hour of robot data**, and *"when GEN-1 adapts to a new task, it is simultaneously adapting to that robot embodiment and to that task for the first time."*

**(b) Quality and mixture beat volume.**
> *"From large-scale ablations, we find that **data quality and diversity matters more than sheer volume**, and that carefully constructed data mixtures can lead to different pretrained model characteristics."*

This is notable coming from the organization with the largest reported corpus — the volume leader publicly disclaiming volume as the governing variable.

## 2. Collection operation (this *is* the data engine)

- **270,000 h** of real-world manipulation trajectories across **thousands of homes, warehouses, and workplaces worldwide**; 500,000+ h by GEN-1.
- **>10,000 new hours per week**, accelerating.
- Powered by *"a global network of hardware and 1,000s of data collection devices and robots"* — low-cost **wearable** devices on humans, not teleoperation rigs.
- Scope ambition: *"every manipulation task humans can think of – from peeling potatoes, to threading bolts – spanning homes, bakeries, laundromats, warehouses, factories."*

**Infrastructure disclosed** (unusually concrete for a blog):
- Custom hardware, dataloaders, and network infrastructure — **including laying new dedicated Internet lines** for uplink bandwidth from global collection sites
- Multi-cloud contracts, custom upload machines
- **O(10K) cores for continual multimodal data processing**
- Dozens of petabytes compressed
- Dataloading techniques from frontier video foundation models, *"capable of absorbing 6.85 years of real-world manipulation experience per day of training"*

## 3. Curation instrument #1 — the semantic browser

> *"an example of searching through <1% of our pretraining dataset… The visualization navigates the user through a **t-SNE map of corresponding language label embeddings** in the dataset. Given a text description, the visualizer locates the nearest neighbor region, and randomly samples in the area a collection of related videos."*

Embedding-space navigation over language labels is a **diversity-auditing tool**: it lets an operator see coverage and holes in a corpus too large to enumerate. Most robotics pipelines have no equivalent; this is standard practice in web-scale text/image curation ported to manipulation.

## 4. Curation instrument #2 — mixture A/B testing across data foundries

The "Science of Pretraining" section is the most distinctive contribution and has no analogue elsewhere in this survey. Data is procured from **multiple data-foundry partners**, and each partner's output is stratified by **collection class**:

| Class | Definition |
|---|---|
| **Class 1** | Data on **specific tasks** |
| **Class 2** | Everything in between (further split, e.g. "Objs" vs "Skills") |
| **Class 3** | **"Do-anything" type data** |

**Eight distinct pretraining datasets** (Partner A Class 1/2/3/2+3, Partner B Class 1/2-Objs/2-Skills, Partner C Class 3) were each used to pretrain a model, then finetuned on **10 long-horizon task sets grouped into three evaluation dimensions — dexterity, real-world applications, generalization** — and scored on two metrics.

> *"Having multiple data collection strategies at scale allows us to **continually A/B test which data improves pretraining the most**… we can use these experiments to evaluate between partners to iterate and provide feedback on **what data to collect, how to do it, and which methods improve models the most**."*

**Curation here is a closed loop back into collection**, not a filter applied after the fact. That is the single most transferable idea in this entry.

### The two-metric diagnostic

Beyond validation MSE, they report **reverse KL divergence**, *"which better measures mode-seeking behavior"*, estimated by Monte Carlo: the policy induces `q = (1/M)Σ N(a; â_m, I)` from M samples, the data induces `p(a) = N(a; a*, I)`, and

```
D̂_KL(q‖p) ≈ (1/M) Σ_m [ log q(â_m) − log p(â_m) ]
```

The diagnostic use is what matters:
> *"models with **both low prediction errors and low reverse KL** tend to perform better with supervised finetuning (SFT) for post-training, while models with **high prediction errors and low reverse KL** tend to be more **distributionally multimodal, which can help post-training reinforcement learning**."*

So the *same* corpus can be good or bad depending on the intended post-training regime — a two-dimensional data-quality readout that lets mixture selection be targeted at SFT vs RL. This is a materially more sophisticated notion of "data quality" than a single scalar score.

## 5. Scaling evidence

**(a) Model-size phase transition / ossification.** In an unprecedented high-data regime:
| Size | Behavior |
|---|---|
| **1B** | *"struggle to absorb complex and diverse sensorimotor data during pretraining – model weights become unable to absorb new information over time"* — clear, early **ossification** |
| **6B** | begins to benefit; strong multi-task capability |
| **7B+** | internalizes large-scale pretraining data, transfers downstream with only a few thousand post-training steps |

> *"To our knowledge, this is the first time that model ossification has been observed in robotics."*

Their explanation invokes Moravec's paradox: ossification appears in LLMs at O(10M) parameters but in robotics at O(1B) — *"intelligence in the physical world may have a higher activation threshold in terms of compute."*

**This is a data-curation finding in disguise:** below ~7B, adding data actively fails. Corpus scale and model scale must be co-designed.

**(b) Pretraining-data power law.**
```
L(D) = (D_c / D)^{α_D}
```
where `D` = pretraining dataset size (in action trajectories) and `L` = downstream validation error after a fixed finetuning budget. Its stated operational use is procurement planning:
> *"we can answer questions like 'how much pretraining data do we need to reach a specific next-action prediction error?' or **'how much post-training data (for a specific task) can we buy with more pretraining data?'**"*

For Clothes Handling they extrapolate to **1 billion action trajectories**. Scaling laws are used here as a *budgeting tool for data collection* — the most practically consequential use of a scaling law in this survey.

**(c) It transfers to real robots, verified blind.** Blind A/B evaluations; models post-trained on **5.6 hours (1%) of task-specific data** already show gains, with peaks up to 99% when full pretraining is paired with all 550+ hours of task-specific data.

**Leakage control, explicitly stated:** *"we ensured no overlap between the pretraining and post-training datasets, which were collected by different people in entirely different environments."*

**(d) GEN-1 outcomes.** >99% success on several tasks (vs GEN-0's 50% on servicing robot vacuums, and much lower from scratch); ~3× faster task completion; ~1 h robot data per task; **10× less task-specific data** than GEN-0 for comparable performance. Endurance runs: kitting auto parts >1 h unattended, 86 consecutive t-shirt folds, 200+ vacuum services, 1,800+ block packs.

## 6. What is not disclosed
- **No cleaning rules.** Filtering, dedup, annotation QC, and rejection rates are never described. The word "quality" appears as an ablation *outcome*, never as a *procedure*.
- Partner identities, per-class hour counts, and the mixture actually used for the shipped model are all withheld.
- Table 1's absolute numbers are tightly clustered (~0.0030–0.0032 prediction error) with no error bars, so the ranking between mixtures is asserted rather than demonstrated.
- No corpus, code, or weights; nothing independently reproducible.
- "Data quality and diversity matters more than sheer volume" is stated as an ablation result but the supporting ablation is not shown in detail.

## 7. Transferable takeaways
1. **Close the loop from evaluation back to collection.** Stratify sources by collection *class*, pretrain one model per stratum, and let downstream results dictate next quarter's collection spec.
2. **Two metrics, not one.** MSE + reverse KL distinguishes "accurate" from "mode-collapsed", and tells you whether a corpus suits SFT or RL post-training.
3. **Use scaling laws as procurement math** — trading pretraining hours against per-task post-training hours is a budget decision that a fitted power law makes explicit.
4. **Build a semantic browser over your corpus.** t-SNE over language-label embeddings makes coverage gaps visible at a scale where enumeration is impossible.
5. **Co-design corpus scale with model scale.** Below the ossification threshold, more data is wasted; GEN-0 puts that threshold near 7B for manipulation.
6. **Enforce collector/environment disjointness** between pretraining and post-training sets, not just episode-level splits.
