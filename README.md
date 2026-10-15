# Every option scored. One forward pass. Open weights.

**SCM turns any text into a probability for each choice, yes/no or score, on a Qwen3 you can host.** Three LoRA sizes (1.7B, 4B, 14B), documented training data, honest benchmarks.

SCM (Sovereign Classification Model) answers typed questions about a text in one forward pass, without generating tokens: choose one of K options, answer a yes/no statement, or pick an ordinal level, with a probability for every option. This repository holds the public documents. It contains no code. See the [positioning](docs/positioning.md) for who it is for, what it replaces, and where it is not the best choice.

## Models (Hugging Face)
| Release | Base | Link |
|---|---|---|
| scm-min-1.7b-v0 | Qwen3-1.7B-Base | https://huggingface.co/5ak3t/scm-min-1.7b-v0 |
| scm-mid-4b-v0 | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v0 |
| scm-max-14b-v0 | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v0 |

Each repository holds a LoRA adapter only, a model card, the training-pool manifests with source licences, and the prompt module. They are experimental research releases.

## Documents
- [Positioning and messaging](docs/positioning.md): who it is for, against what, and where it is not the best choice.
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
