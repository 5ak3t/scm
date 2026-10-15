# Rated typed decisions from open Qwen3 models

**Give SCM a text and a question, and get a probability for every answer.** Pick one of several options, answer yes or no, or place the text on a scale.

SCM (Sovereign Classification Model) is a family of three open models built on Qwen3, at 1.7, 4 and 14 billion parameters. This repository holds the public documents. It contains no code.

## Why SCM

The best typed-decision models today are very good and very large. The hosted Jev service is the accuracy reference, and the strongest open models sit around 27 billion parameters. Each asks you to accept something. With Jev your text goes to someone else's servers, every call is billed, and you cannot run the model yourself. With the 27-billion-parameter open models you need serious hardware to run them at all. At the other end, very small specialised models are fast but give up accuracy, and some are trained on the benchmark's own data families, which makes their scores hard to compare.

SCM sits between the two and is built to be inspected. There are three sizes. The 14-billion model has about half the parameters of the 27-billion class, and the 4 and 1.7-billion models fit modest hardware. Everything about them is open: the model files, every training source with its licence, the sources we kept out and why, and an audit of how much of a public benchmark overlaps our training data, published next to the results. Options are given with each question, so a new workflow needs a new list of options, not a new training run.

The evidence is modest and we state it plainly. On the same data, accuracy on questions from workflows the model has never seen climbs from 50.5 at 0.6 billion parameters to 81.0 at 14 billion, so size is a real lever. On the public S1MB English benchmark the 14-billion model completes 131 of 137 benchmarks, with a full disclosure of training overlap.

SCM is not the most accurate typed-decision model. On the same benchmark it scores below the hosted Jev 1.13 reference on yes/no questions (50.1 against 64.6), on choice questions (52.9 against 65.2) and on score questions (32.3 against 46.9), and below the 27-billion class. It gives plain probabilities with no calibration or error guarantee, handles at most 20 options per question, and some training sources are non-commercial or share-alike. If accuracy is all that matters, use the leaders. If you want an open, self-hostable model whose data and evaluation you can check, and you are willing to test it on your own data first, that is what SCM is for.

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
  title        = {SCM: rated typed decisions from open Qwen3 models},
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
