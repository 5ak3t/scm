# Positioning and messaging canvas

Framework: the Positioning & Messaging Canvas (use-case-based positioning, then supporting messaging per competitive alternative). Every claim below is one we can back with a number or a document in this repository; where SCM is not the best choice, the canvas says so.

## Positioning strategy summary
**SCM is taking a use-case-based positioning strategy: open, one-pass typed decisions for teams that route, triage and gate text inside their own workflows.**

| Target organisation | Target roles | Product category | Primary use case |
|---|---|---|---|
| Teams that host their own models: startups and mid-size product, data and platform teams, plus researchers working on typed decisions | **Primary:** ML / AI engineer, platform engineer. **Secondary:** head of data, product owner of an automation | Open typed-decision models (LoRA adapters on Qwen3) | Turn a text into a probability for every option of a question (pick one, yes/no, ordinal score) in one forward pass, on infrastructure you control, with options that change per workflow |

| Competitive alternative | Problem summary | Differentiation summary |
|---|---|---|
| **Prompting a general LLM and parsing its answer** (also: per-workflow classifiers, hosted typed-decision APIs, larger open typed-decision models) | Teams need an answer from a fixed set of options, with a usable confidence, thousands of times a day. Generated answers are slow and fragile to parse and give no per-option probabilities. Classifiers need labelled data and a retrain when the options change. Hosted APIs send your text out and bill per call. The strongest open models are heavy to host. | SCM scores **every option of a question in a single forward pass** with a LoRA on a Qwen3 base you can host. Options are supplied at run time. Three sizes (1.7B, 4B, 14B) trade cost for accuracy. Weights are open and the training data, licences and benchmark overlap are documented. It is **not the most accurate typed-decision model**; see "Where SCM is not the best choice". |

## Supporting messaging

| Competitive alternative (details) | Why that sucks (problems) | Capabilities (how we do it differently) | Features | Benefits (why that's better) |
|---|---|---|---|---|
| **1. Prompt a general LLM, then parse the text** | Generation is slow and its length varies. Answers need parsing and sometimes fail to parse. There is no per-option probability to threshold or rank on. | One forward pass, no generation. A probability for every option. | Lettered-option prompt; next-token letter logits; scores averaged over four option orders; 1.7B, 4B and 14B adapters. | Predictable cost per question, no parsing step, and a probability per option to rank and route on. (Raw softmax: validate thresholds on your own data.) |
| **2. Train a classifier per workflow** | Needs labelled data for every workflow. A new or changed option list means a retrain. Each classifier handles only its own label set. | Options are supplied at run time, for choice, yes/no and ordinal score alike. One adapter covers many workflows. | Trained on 124,334 records from 168 public workflow groups; typed questions with option text in the prompt. | Add a workflow by writing its options, not by training a model. On unseen workflows accuracy is 60 to 81% on knowledge multiple choice depending on size, so validate before relying on it. |
| **3. Hosted typed-decision APIs and larger open models** | Your text leaves your infrastructure and every call is billed. The 27B class needs heavy GPUs. Closed weights cannot be inspected or fine-tuned. | Open LoRA adapters you can host, merge into a fp16 base and inspect. Choose the size that fits your budget. | Adapter on an Apache-2.0 Qwen3 base; pinned base revisions; documented sources, licences and exclusions; published overlap audit. | Self-hostable, auditable and cheaper to run. You give up accuracy to the strongest hosted and 27B-class models (see below). |

## Proof points (all measured)
- **Scales with size:** unseen knowledge multiple choice 50.5, 60.5, 71.9, 76.4, 81.0 for 0.6B, 1.7B, 4B, 8B, 14B on the same data (single seed). See [results](results.md).
- **External benchmark:** S1MB English v1, `scm-max-14b-v0`: 131 of 137 benchmarks complete, noul 50.1, score 32.3, gen-choice 96.7 (baseline-adjusted). Disclosure and overlap audit: [S1MB submission](s1mb-submission.md).
- **Transparent data:** every training source with its recorded licence, the synthetic tickets, and the exclusions are listed in [data and licences](data-and-licences.md).

## Where SCM is not the best choice
- You need the highest accuracy on the S1MB-style benchmarks: `scm-max-14b-v0` scores below the hosted Jev 1.13 reference on noul (50.1 against 64.6), choice (52.9 against 65.2 on 51 benchmarks) and score (32.3 against 46.9), and below the 27B-class entries.
- You need a coverage or error guarantee: the released weights ship raw probabilities, no calibration and no abstention.
- Your questions have more than 20 options, or very long inputs: the adapters were not trained for either.
- You need a commercial licence with no review: some training sources are non-commercial or share-alike.

## Messaging
**Headline:** Every option scored. One forward pass. Open weights.

**Subhead:** SCM turns any text into a probability for each choice, yes/no or score, on a Qwen3 you can host. Three LoRA sizes. Documented training data. Honest benchmarks.

**Taglines**
- Ask once. Get a probability for every option.
- Typed decisions you can run, read and audit.
- From 1.7B to 14B: pick the size, keep the weights.

**Boilerplate (one line):** SCM is a family of open LoRA adapters (1.7B, 4B, 14B) on Qwen3 that score every option of a typed question in one forward pass.

**Why not "calibrated" in the headline:** the released probabilities are an uncalibrated softmax. We will use that word when a calibration layer ships with a measured coverage result.
