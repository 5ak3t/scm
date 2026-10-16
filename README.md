# Rated typed decisions

**Give SCM a text and a question, and get a probability for every answer.** Pick one of several options, answer yes or no, or place the text on a scale.

SCM (Sovereign Classification Model) is a family of open models built on Qwen3, at 1.7, 4 and 14 billion parameters. **Version 1 (v1) is the current release**: a 14-billion and a 4-billion model. The 1.7-billion v1 model did not pass our release check and is not published (its v0 stays available). This repository holds the public documents. It contains no code.

## Why SCM

The best typed-decision models today are very good and very large. The hosted Jev service is the accuracy reference, and the strongest open models sit around 27 billion parameters. Each asks you to accept something. With Jev your text goes to someone else's servers, every call is billed, and you cannot run the model yourself. With the 27-billion-parameter open models you need serious hardware to run them at all. At the other end, very small specialised models are fast but give up accuracy, and some are trained on the benchmark's own data families, which makes their scores hard to compare.

SCM sits between the two and is built to be inspected. There are three sizes. The 14-billion model has about half the parameters of the 27-billion class, and the 4 and 1.7-billion models fit modest hardware. Everything about them is open: the model files, every training source with its licence, the sources we kept out and why, and an audit of how much of a public benchmark overlaps our training data, published next to the results. Options are given with each question, so a new workflow needs a new list of options, not a new training run.

The evidence is modest and we state it plainly. On the same data, accuracy on questions from workflows the model has never seen climbs from 50.5 at 0.6 billion parameters to 81.0 at 14 billion, so size is a real lever. On the public S1MB English benchmark the 14-billion v1 model completes all 137 benchmarks and scores 55.5 (v0: 45.9), with a full disclosure of training overlap. Part of that gain comes from training on the training splits of the datasets S1MB is built from, so it is not a zero-shot number (see below).

SCM is not the most accurate typed-decision model. On the same benchmark the 14-billion v1 model scores below the hosted Jev 1.13 reference on yes/no questions (60.5 against 64.6), score questions (39.2 against 46.9) and, narrowly, choice questions (66.7 against 67.2), and below the 27-billion class. It gives plain probabilities with no calibration or error guarantee, handles up to 255 options per question in v1 (20 in v0), and some training sources are non-commercial or share-alike. If accuracy is all that matters, use the leaders. If you want an open, self-hostable model whose data and evaluation you can check, and you are willing to test it on your own data first, that is what SCM is for.

## Results

All figures are accuracy-style scores in percent (higher is better). The size table uses our own open test sets and one training run per size, so treat small gaps as noise. The S1MB table is an outside benchmark.

**Size comparison (same data and recipe)**

| Model | Questions from unseen workflows | Five in-house decision suites | Security triage (accuracy) | Security triage (balanced) |
|---|---|---|---|---|
| scm-min-1.7b-v0 | 60.5 | 66.3 | 57.0 | 40.4 |
| scm-mid-4b-v0 | 71.9 | 72.1 | 57.8 | 35.9 |
| scm-max-14b-v0 | **81.0** | **76.4** | 64.5 | **54.0** |

On security triage, always answering "open an investigation" scores 71.3, so read the balanced column. The full table, including the 0.6B and 8B runs we did not publish, is in [docs/results.md](docs/results.md).

**S1MB English v1 (baseline-adjusted score, 0 to 100, all 137 benchmarks complete, zero failed decisions)**

| | Yes/no (59) | Choice (57) | Score (21) | Task average | Rank* |
|---|---|---|---|---|---|
| scm-max-14b-v1 | 60.5 | 66.7 | 39.2 | 55.5 | 19 of 165 |
| scm-mid-4b-v1 | 45.9 | 38.2 | 6.3 | 30.2 | 94 of 165 |
| scm-max-14b-v0 (earlier release) | 50.1 | 55.4 | 32.3 | 45.9 | 45 of 164 |
| Jev 1.13 (reference) | 64.6 | 67.2 | 46.9 | 59.6 | 17 of 165 |

*Our own computation from the saved results, ordering by the leaderboard's Borda rule. The shared leaderboard may order differently. v1 results (14B) are in an open results pull request awaiting review: https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/38. The v0 results are in https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/37.

**Read the v1 numbers with this in mind.** v1 was trained on the training splits of 65 dataset families that S1MB benchmarks are built from (79 declared "same source family" entries), after removing inputs that matched S1MB test inputs. An audit of the final training pool still finds 304 of 15,609 English test cases with an approximate match, mostly in templated families. These are candidates, not confirmed use of test inputs, and no exact use is known. Compare with models that did not train on these families with care. Details: [docs/s1mb-submission.md](docs/s1mb-submission.md).

![S1MB task average against model size](docs/figures/s1mb-v1.svg)

![Accuracy against model size](docs/figures/scaling.svg)

## Models (Hugging Face)
| Release | Base | Link |
|---|---|---|
| scm-max-14b-v1 (current) | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v1 |
| scm-mid-4b-v1 (current) | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v1 |
| scm-min-1.7b-v0 | Qwen3-1.7B-Base | https://huggingface.co/5ak3t/scm-min-1.7b-v0 |
| scm-mid-4b-v0 | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v0 |
| scm-max-14b-v0 | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v0 |

Each repository holds the model weights (a small add-on to the Qwen3 base model), a model card, the training-data lists with source licences, and the prompt format. They are experimental research releases. There is no 1.7B v1: it was trained but scored below its v0 on our held-out release check ([what v1 changed](docs/v1.md)).

## Documents
- [What v1 changed](docs/v1.md): training data, release check, readout experiments that did not help.
- [Results](docs/results.md): size ladder, charts, S1MB summary.
- [How to use the models](docs/usage.md): how to ask a question, how the answer is scored, the 20-option limit.
- [Training data and licences](docs/data-and-licences.md): sources by group, recorded licences, synthetic tickets, exclusions.
- [Intended use and limits](docs/intended-use.md).
- [S1MB submission](docs/s1mb-submission.md) and the [overlap report](docs/s1mb-overlap-report.json).
- [Changelog](CHANGELOG.md).

## Status
Selection-level results on open development data, mostly single-seed. The 14B v1 model is below the strongest S1MB entries (rank 19 of 165 by our computation, with the training-overlap caveat above). Read the intended-use page before relying on these weights.

## Cite
```bibtex
@misc{scm2026,
  title        = {SCM: rated typed decisions},
  author       = {5ak3t},
  year         = {2026},
  howpublished = {\url{https://github.com/5ak3t/scm}},
  note         = {Version 1}
}
```
A `CITATION.cff` is included.

## Contact
Report issues at https://github.com/5ak3t/scm/issues (or the Community tab of the Hugging Face repositories).

## Licence
The documents and figures in this repository are CC BY 4.0 (see `LICENSE`). The model weights are separate and have their own terms; see [data and licences](docs/data-and-licences.md).
