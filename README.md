# Rated typed decisions

**Give SCM a text and a question, and get a probability for every answer.** Pick one of several options, answer yes or no, or place the text on a scale.

SCM (Sovereign Classification Model) is a family of three open models built on Qwen3, at 1.7, 4 and 14 billion parameters. This repository holds the public documents. It contains no code.

## Why SCM

The best typed-decision models today are very good and very large. The hosted Jev service is the accuracy reference, and the strongest open models sit around 27 billion parameters. Each asks you to accept something. With Jev your text goes to someone else's servers, every call is billed, and you cannot run the model yourself. With the 27-billion-parameter open models you need serious hardware to run them at all. At the other end, very small specialised models are fast but give up accuracy, and some are trained on the benchmark's own data families, which makes their scores hard to compare.

SCM sits between the two and is built to be inspected. There are three sizes. The 14-billion model has about half the parameters of the 27-billion class, and the 4 and 1.7-billion models fit modest hardware. Everything about them is open: the model files, every training source with its licence, the sources we kept out and why, and an audit of how much of a public benchmark overlaps our training data, published next to the results. Options are given with each question, so a new workflow needs a new list of options, not a new training run.

The evidence is modest and we state it plainly. On the same data, accuracy on questions from workflows the model has never seen climbs from 50.5 at 0.6 billion parameters to 81.0 at 14 billion, so size is a real lever. On the public S1MB English benchmark the 14-billion model completes 131 of 137 benchmarks, with a full disclosure of training overlap.

SCM is not the most accurate typed-decision model. On the same benchmark it scores below the hosted Jev 1.13 reference on yes/no questions (50.1 against 64.6), on choice questions (52.9 against 65.2) and on score questions (32.3 against 46.9), and below the 27-billion class. It gives plain probabilities with no calibration or error guarantee, handles at most 20 options per question, and some training sources are non-commercial or share-alike. If accuracy is all that matters, use the leaders. If you want an open, self-hostable model whose data and evaluation you can check, and you are willing to test it on your own data first, that is what SCM is for.

## Results

All figures are accuracy-style scores in percent (higher is better). The size table uses our own open test sets and one training run per size, so treat small gaps as noise. The S1MB table is an outside benchmark.

**Size comparison (same data and recipe)**

| Model | Questions from unseen workflows | Five in-house decision suites | Security triage (accuracy) | Security triage (balanced) |
|---|---|---|---|---|
| scm-min-1.7b-v0 | 60.5 | 66.3 | 57.0 | 40.4 |
| scm-mid-4b-v0 | 71.9 | 72.1 | 57.8 | 35.9 |
| scm-max-14b-v0 | **81.0** | **76.4** | 64.5 | **54.0** |

On security triage, always answering "open an investigation" scores 71.3, so read the balanced column. The full table, including the 0.6B and 8B runs we did not publish, is in [docs/results.md](docs/results.md).

**S1MB English v1: scm-max-14b-v0 against the Jev 1.13 reference** (baseline-adjusted score, 0 to 100)

| | Yes/no (59 benchmarks) | Choice (51 of 57) | Score (21) | Synthetic yes/no | Synthetic choice | Synthetic score |
|---|---|---|---|---|---|---|
| scm-max-14b-v0 | 50.1 | 52.9 | 32.3 | 88.0 | 96.7 | 74.1 |
| Jev 1.13 | 64.6 | 65.2 | 46.9 | 99.0 | 98.7 | 91.1 |

131 of 137 benchmarks completed. The six that did not (questions with more than 20 options) are saved as failures, so the shared leaderboard shows no choice total for SCM. The 3 typed-decisions benchmarks overlap our training data; the details are in [docs/s1mb-submission.md](docs/s1mb-submission.md). Results pull request: https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/37

![Accuracy against model size](docs/figures/scaling.svg)

## Models (Hugging Face)
| Release | Base | Link |
|---|---|---|
| scm-min-1.7b-v0 | Qwen3-1.7B-Base | https://huggingface.co/5ak3t/scm-min-1.7b-v0 |
| scm-mid-4b-v0 | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v0 |
| scm-max-14b-v0 | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v0 |

Each repository holds the model weights (a small add-on to the Qwen3 base model), a model card, the training-data lists with source licences, and the prompt format. They are experimental research releases.

## Documents
- [Results](docs/results.md): size ladder, charts, S1MB summary.
- [How to use the models](docs/usage.md): how to ask a question, how the answer is scored, the 20-option limit.
- [Training data and licences](docs/data-and-licences.md): sources by group, recorded licences, synthetic tickets, exclusions.
- [Intended use and limits](docs/intended-use.md).
- [S1MB submission](docs/s1mb-submission.md) and the [overlap report](docs/s1mb-overlap-report.json).
- [Changelog](CHANGELOG.md).

## Status
Selection-level results on open development data, mostly single-seed. The 14B model is well below the strongest S1MB entries. Read the intended-use page before relying on these weights.

## Cite
```bibtex
@misc{scm2026,
  title        = {SCM: rated typed decisions},
  author       = {5ak3t},
  year         = {2026},
  howpublished = {\url{https://github.com/5ak3t/scm}},
  note         = {Version 0}
}
```
A `CITATION.cff` is included.

## Contact
Report issues at https://github.com/5ak3t/scm/issues (or the Community tab of the Hugging Face repositories).

## Licence
The documents and figures in this repository are CC BY 4.0 (see `LICENSE`). The model weights are separate and have their own terms; see [data and licences](docs/data-and-licences.md).
