# Every option scored. One forward pass. Open weights.

**SCM turns any text into a probability for each choice, yes/no or score, on a Qwen3 you can host.** Three LoRA sizes (1.7B, 4B, 14B), documented training data, honest benchmarks.

SCM (Sovereign Classification Model) answers typed questions about a text in one forward pass, without generating tokens: choose one of K options, answer a yes/no statement, or pick an ordinal level, with a probability for every option. This repository holds the public documents. It contains no code.

## Why SCM

Most teams that need a decision from text do it the slow way. They prompt a general LLM, wait for it to write an answer, and parse what comes back. There is no probability per option to rank or threshold on, and the parser breaks on the day the model phrases something differently. Others train a classifier for every workflow, which needs labelled data and a retrain each time the list of options changes. Hosted typed-decision APIs are accurate but send your text out and bill every call, and the strongest open models are heavy to host.

SCM is for engineers who run their own models and want something simpler. You give it a text and a question with its options, and it scores every option in a single forward pass. The options are supplied at run time, so a new workflow means writing a new option list, not training a model. It is a LoRA on a Qwen3 base, so you can host it, merge it into a fp16 base, and read every training source and licence behind it. Pick the size that fits your budget: 1.7B, 4B or 14B.

The evidence is modest and we state it plainly. On the same data, accuracy on unseen knowledge multiple choice goes 50.5, 60.5, 71.9, 76.4 and 81.0 from 0.6B to 14B, so size is a real lever. On the external S1MB English benchmark the 14B model completes 131 of 137 benchmarks and scores 50.1 on yes/no, 32.3 on ordinal score and 96.7 on the synthetic generalization choice subsets, with a full training-overlap disclosure.

SCM is not the best typed-decision model. The 14B model scores below the hosted Jev 1.13 reference on yes/no (50.1 against 64.6), choice (52.9 against 65.2 on the 51 complete benchmarks) and score (32.3 against 46.9), and below the 27B-class entries. The released weights give raw probabilities with no calibration, no abstention and no coverage guarantee. They handle at most 20 options and are not trained for very long inputs, and some training sources are non-commercial or share-alike. If you need the highest accuracy or a guarantee, use something else. If you want open, self-hosted, auditable one-pass decisions and are willing to validate them on your own data, this is what SCM is for.

**Ask once. Get a probability for every option.** Typed decisions you can run, read and audit. From 1.7B to 14B: pick the size, keep the weights.

## Models (Hugging Face)
| Release | Base | Link |
|---|---|---|
| scm-min-1.7b-v0 | Qwen3-1.7B-Base | https://huggingface.co/5ak3t/scm-min-1.7b-v0 |
| scm-mid-4b-v0 | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v0 |
| scm-max-14b-v0 | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v0 |

Each repository holds a LoRA adapter only, a model card, the training-pool manifests with source licences, and the prompt module. They are experimental research releases.

## Documents
- [Results](docs/results.md): size ladder, charts, S1MB summary.
- [How to use the models](docs/usage.md): prompt format, option letters, order averaging, the 20-option limit.
- [Training data and licences](docs/data-and-licences.md): sources by group, recorded licences, synthetic tickets, exclusions.
- [Intended use and limits](docs/intended-use.md).
- [S1MB submission](docs/s1mb-submission.md) and the [overlap report](docs/s1mb-overlap-report.json).
- [Changelog](CHANGELOG.md).

## Status
Selection-level results on open development data, mostly single-seed. The 14B model is well below the strongest S1MB entries. Read the intended-use page before relying on these weights.

## Cite
```bibtex
@misc{scm2026,
  title        = {SCM: open, one-pass typed decisions from a Qwen3 LoRA},
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
The documents and figures in this repository are CC BY 4.0 (see `LICENSE`). The model adapters are separate artefacts with their own terms; see [data and licences](docs/data-and-licences.md).
