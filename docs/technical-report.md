# SCM: calibrated typed decisions from a Qwen3 LoRA in one forward pass

Technical report, version 0 (2026-10-15). Models: `scm-min-1.7b-v0`, `scm-mid-4b-v0`, `scm-max-14b-v0` (Hugging Face, user `5ak3t`). Numbers are selection-level results on open development data unless stated otherwise.

## Abstract

SCM (Sovereign Classification Model) answers typed questions about a state in one forward pass, without decoding tokens: `choice` (pick one of K options supplied at run time), `noul` (a yes/no statement) and `score` (an ordered level). It is a Qwen3 base with a LoRA adapter that scores all options of a question in a single prompt as lettered lines and reads the next-token logit of each letter; several option orders are averaged. We release three adapters (1.7B, 4B, 14B) trained on 124,334 records from 168 public workflow groups. On a held-out knowledge multiple-choice set, accuracy rises with model size on identical data (.719, .764, .810 for 4B, 8B, 14B), while adding more workflow groups does not help (.733, .714, .719 for 12, 25, 168 groups). A static conformal layer needs set size to keep coverage on new workflows (90% coverage at 3.2 of 4 options). We also document 48 ideas that did not help ([what-did-not-work.md](what-did-not-work.md)). The models are an open experimental release, not a state-of-the-art claim; no pre-registered keep rule was fully met by any size.

## 1. Task and interface

A request is a `state` (free text, or a JSON object serialised as text) plus a list of typed questions. Each question has instructions and options:
- `choice`: K option texts supplied at run time; the answer is a distribution over the K options.
- `noul`: exactly two options, `false` and `true`, with authored meanings.
- `score`: ordered levels with numeric values; the answer is a distribution over levels, and its expectation is the score.

The output is one probability vector per question, validated to sum to 1 within 1e-9 (1e-6 in the Python scorers). The motivating use is routing and triage inside workflows (ticket handling, security alerts, agent-trace review), where a calibrated answer in one pass is cheaper than a generated one and where the application needs a confidence it can act on.

## 2. Method

### 2.1 Two scoring formats
- **Per-option (reranker) format.** Each option is scored on its own as a Qwen3-Reranker yes/no pair, `s = logit(yes) - logit(no)`; the vector of option scores goes through a softmax. Permuting the option order cannot change any score. Our first released SCM (4B, LoRA on Qwen3-Reranker-4B) uses this format.
- **Joint-option format (the released v0 family).** All options of one question are shown in one prompt:

```
The following is a multiple choice question. Choose the single best answer.

<instructions>

Question: <state>
A. <option text>
B. <option text>
...
Answer:
```

The score of option i is the next-token logit of its letter (at most 20 options, letters A to T). At test time we score 2 to 4 option orders (identity, reverse, cyclic shifts), average the per-order log-softmax over the shown letters, and normalise. Training uses a random option order per example, which makes the scorer order-robust.

The two formats trade off: joint letters win on unseen knowledge multiple choice (+.17 on a held-out set, two seeds, two recipes) and on most development suites, and lose on one security-triage suite (accuracy .47 to .58 against .70). We therefore describe the choice as a measured trade-off, not a free improvement.

### 2.2 Model family
| Release | Base | LoRA | Notes |
|---|---|---|---|
| `scm-min-1.7b-v0` | Qwen3-1.7B-Base | r16, alpha 32 | below our own dev thresholds |
| `scm-mid-4b-v0` | Qwen3-4B-Base | r16, alpha 32 | fp16, lr 1e-4 |
| `scm-max-14b-v0` | Qwen3-14B-Base | r16, alpha 32 | bf16, lr 5e-5, non-finite-gradient guard |

Adapters are applied to all attention and MLP projections. Base revisions are pinned in each adapter's `adapter_config.json`. Only the adapter is distributed; the base weights keep their own licence (Apache-2.0).

### 2.3 Training
One epoch over pool v5 (3,733 steps of 16 questions), prompts left-truncated at 1,024 tokens, AdamW (weight decay 0), 3% linear warmup then cosine decay, LoRA weights in fp32, random option order, at most 500 questions per workflow group per epoch. Training the 8B and 14B models in fp16, or in bf16 without a guard, diverged (non-finite gradients, NaN by step 40, one run degrading mid-training). A guard that skips any step whose gradient is not finite (all-reduced across ranks) fixed it: 58 of 3,733 steps were skipped for the 8B and 14B runs, 23 for the 0.6B and 1.7B bf16 runs. The 14B run took 7,500 s on two H100 GPUs.

### 2.4 Uncertainty: conformal prediction sets
SCM returns prediction sets with a coverage target (default 90%): split-conformal sets with LAC or mass-adaptive (randomised APS) scores, Mondrian by question type, and workflow-level thresholds (a group-quantile level beta of .2 to .3) validated by leave-suite-out replay. We do not ship an abstention flag (section 6, Abstention). The guarantee is marginal and holds near the calibration data; it is not a source-free guarantee under shift to a harder workflow.

### 2.5 Serving
A reference GPU scorer reproduces the evaluation numbers (maximum absolute difference .0234 on choice and .0156 on noul and score against the evaluation kernels, argmax agreement on 240 of 240 questions). The serving engine and training code are not part of this release.

## 3. Data

Training uses public datasets only, recorded with their licences (not gated): pool v3 (58,091 records, 12 groups), v4 (76,414, 25 groups), and v5 (124,334 records, 168 workflow groups, at most 500 records per group). v5 adds topic, sentiment, NLI, multiple-choice reasoning, preference, legal, spam and security-adjacent sources. The in-house parts of earlier pools are 780 synthetic support-ticket records written by an LLM (Claude Haiku) from deterministic specs and 724 typed-decision examples. No labels were generated with Claude, GPT or another LLM for pool v5 or for these adapters. Several public sources carry annotations produced by their authors' LLMs (for example UltraFeedback). A reserved list excluded evaluation sources (MMLU, ARC, CommonsenseQA, MedMCQA, ScienceQA, SNLI, SciTail and others) from training; contamination checks dropped 546 of 48,466 candidate records. Pool manifests (`pool-v{3,4,5}-manifest.json`) are published with the adapters; raw records are not.

## 4. Evaluation protocol

Development suites (`zs_wide`, `zs_td`, `td`, `zs_massive`, `tickets`) and the open half of a fresh unseen-workflow set (`fresh_sel`, knowledge multiple choice) select models; sealed halves and finals are read once. Pre-registered keep rules: fresh-set accuracy at least .628 (K1), five-suite dev total at least .7091 (K2), security-triage accuracy at least .6513 (K3). K3 turned out to be a weak yardstick because the majority answer scores .713; balanced accuracy is used from now on. Intervals are paired or stratified-bootstrap 95% intervals clustered by record.

## 5. Results

### 5.1 Size ladder (same data and recipe family, seed 0; open selection data)
| Model | fresh_sel | dev (5 suites) | zs_td acc | zs_td balanced | sec_cve_open |
|---|---|---|---|---|---|
| shipped per-option 4B | .548 | .7291 | .7013 | .409 | n/a |
| `scm-min` 1.7B | .605 | .663 | .570 | .404 | .505 |
| 0.6B | .505 | .607 | .506 | .328 | .466 |
| `scm-mid` 4B | .719 | .721 | .578 | .359 | .425 |
| 8B | .764 | .737 | .604 | .430 | .553 |
| `scm-max` 14B | **.810** | **.764** | .645 | **.540** | **.598** |

Each doubling from 4B adds about +.045 on unseen knowledge multiple choice. The 14B passes K1 and K2 and misses K3 by .006 (.645 against .6513); no model passes K1 to K3, so under the pre-registered rule no ladder arm is a "kept candidate" and the shipped per-option SCM stays the benchmarked reference. We publish the three adapters anyway as an open experimental release. Numbers are single-seed.

### 5.2 Sealed finals of the first release (per-option 4B, 12 models)
On v2 finals (read once): most accurate on `tickets` (.7554), lowest NLL on `td`, `tickets` and `tickets_ood`, third of 12 on the unseen-workflow suite `zs_wide2` (macro .630; hosted Jev .828 and kev-4b .687 above it), `zs_massive2` .921, `zs_td` .664, `td` .723 (below the .78 goal). Of six pre-registered targets, 3 were met, 1 half met, 2 missed. Pooled conformal coverage on the finals was .869 against a .90 target.

### 5.3 Calibration on genuinely new workflows
Sealed half of a fresh set: v1 coverage **.705**; v2 (workflow-level thresholds) **.841** against .88 required; v3 (mass-adaptive sets, beta .3, no abstention) **.9015 [.870, .930]** at **3.2 options per 4-option question**. A confidence-only shift detector did not beat the static rule (.864, .860, .874 against .862); only a withhold rule helped modestly. A coverage guarantee under such shift would need a few labelled examples from the new workflow (few-label recalibration); this is not implemented.

### 5.4 Triage and prior correction
On security triage the best possible per-option prior correction gives balanced accuracy .32 to .34 for every model, i.e. the model collapses to the majority answer; the joint-option models' failure there is a weak within-workflow ranking, not a fixable prior.

### 5.5 S1MB English v1
We evaluated `scm-max-14b-v0` (revision `1e983c1a1a386b9a49f04fa219e5b1ff8cd12058`) on all 137 S1MB English benchmarks (one run, one H100, 3,589 s) with a configuration fixed before any S1MB score: fp16 merged adapter, 4 option orders, no calibration or temperature, a 16,384-token input limit with overflow rejected, and at most 20 candidates. 131 benchmarks completed. Six (banking77, clinc, gretel-pii, hwu64, massive, mtop) have questions with more than 20 candidates, so all 651 of their cases were rejected (1,104 failed decisions); nothing else failed, and no input-length or numeric error occurred. The submission is therefore partial and does not qualify for aggregates that need complete coverage.

Baseline-adjusted scores (0 to 100, equal benchmark weights), against the pinned Jev 1.13 reference results:

| | Noul (59) | Choice (51 of 57 complete) | Score (21) | gen-noul | gen-choice | gen-score |
|---|---|---|---|---|---|---|
| scm-max-14b-v0 | 50.09 | 52.9 | 32.31 | 88.00 | 96.71 | 74.07 |
| Jev 1.13 (same benchmarks) | 64.63 | 65.2 | 46.92 | 99.00 | 98.68 | 91.14 |

The repository's choice aggregate is N/A because of the six incomplete benchmarks; 52.9 and 65.2 are our means over the 51 complete choice benchmarks. SCM-max is well below the reference model and below the strongest published entries. The three `laya--typed-decisions` benchmarks are inflated by training overlap (section 8). The results were submitted as a pull request to the shared results dataset (`hotchpotch/s1mb-result`, discussion 37).

## 6. What did not work (summary)

48 ideas are documented with numbers in [what-did-not-work.md](what-did-not-work.md). The ones that shaped the design:
1. Label-free noise-floor (batch) calibration lowered accuracy by up to -.149; it removes the true label prior when labels within a template are skewed.
2. A trained head on frozen 8B features never beat zero-shot; LoRA fine-tuning did.
3. A 0.6B student distilled from an 8B teacher lost unseen-workflow robustness (-.106 on security triage).
4. More workflow groups (12, 25, 168) did not improve unseen accuracy; broader generic data hurt the small model.
5. Static conformal calibration misses coverage on new workflows, and risk-controlled abstention had no certifiable threshold on choice and was withdrawn.
6. Fit-to-fit noise on a single held-out workflow is about .05 accuracy.

### Abstention
The older release had risk-controlled abstention thresholds. Refitting on the larger calibration set removed them (choice: no certifiable threshold; noul: out-of-fold selective accuracy .922 against the .95 budget), so the shipped calibration contains prediction sets only.

## 7. Limitations
- Selection-level numbers on open data, mostly single-seed; the ladder has three comparable sizes and no variance.
- Accuracy on workflows unlike the training data is far below in-distribution accuracy (.55 to .81 on unseen knowledge multiple choice, depending on size).
- Security triage is weak for every joint model; plain accuracy on that suite is a poor yardstick.
- Prompts longer than about 1,024 tokens are out of distribution (training truncation); S1MB inputs go up to about 14.8k tokens.
- At most 20 options per question in the joint format.
- The coverage guarantee holds near the calibration data only; no abstention.
- The pre-registered keep rules were not all met; K3 was later found to be weak.

## 8. Training-exposure and overlap disclosure

We audited pool v5 against the S1MB English test inputs (inputs only; a case overlaps when its normalised state matches a pool state, or when at least half of its 8-word shingles occur in the pool; the report is `s1mb-overlap-report.json` in the model repository). 101 test cases overlap, concentrated in `laya--typed-decisions-{choice,noul,score}-test-v1` (30 of 100 each, the same states), because pool v3 contains the `LocalLLaMA/typed-decisions` train split (724 records); we declare these as evaluation-example overlap. Further source families present in training and in S1MB English (banking77, clinc, dbpedia, go_emotions, AG News, Enron spam, PAWS, OpenBookQA, AQuA-RAT, Civil Comments, HH-RLHF, ContractNLI via LegalBench) are declared as same-source-family. The Bekko and Saihai training datasets were not used, and no S1MB benchmark data was used for training, prompts, option orders, limits or checkpoint selection. The Qwen3 pretraining data is not audited.

## 9. Artifacts
- Adapters: https://huggingface.co/5ak3t/scm-min-1.7b-v0, https://huggingface.co/5ak3t/scm-mid-4b-v0, https://huggingface.co/5ak3t/scm-max-14b-v0 (adapter, pinned base revision, model card, pool manifests with source licences, prompt module; the 14B repo also holds the S1MB adapter patch and the overlap report).
- S1MB results: https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/37.
- Naming: releases are `scm-{min|mid|max}-{params}-v0`.

## 10. Licences and use
The adapters are released for research. The base models are Apache-2.0. The training data carry the licences recorded in the pool manifests; check them before commercial use. The 780 synthetic tickets are in-house, generated from deterministic specifications.

## References
Hu et al., LoRA (2021). Dettmers et al., QLoRA (2023). Sadinle et al., least-ambiguous set-valued classifiers (2019). Romano et al., adaptive prediction sets (2020). Angelopoulos et al., Learn then Test (2021). Zheng et al., large language models are not robust multiple choice selectors (2023). Qwen team, Qwen3 technical report (2025). S1MB, the System One Mosaic Benchmark (https://github.com/hotchpotch/S1MB).
