# SCM: calibrated typed decisions from a Qwen3 LoRA

SCM (Sovereign Classification Model) answers typed questions about a state in one forward pass, without generating tokens: choose one of K options, answer a yes/no statement, or pick an ordinal level, with a probability for every option. This repository holds the public reports. It contains no code.

## Models (Hugging Face)
| Release | Base | Link |
|---|---|---|
| scm-min-1.7b-v0 | Qwen3-1.7B-Base | https://huggingface.co/5ak3t/scm-min-1.7b-v0 |
| scm-mid-4b-v0 | Qwen3-4B-Base | https://huggingface.co/5ak3t/scm-mid-4b-v0 |
| scm-max-14b-v0 | Qwen3-14B-Base | https://huggingface.co/5ak3t/scm-max-14b-v0 |

Each repo holds a LoRA adapter only, a model card, the training-pool manifests with source licences, and the prompt module. They are experimental research releases.

## Documents
- [docs/technical-report.md](docs/technical-report.md): method, data, results, limitations, disclosure.
- [docs/what-did-not-work.md](docs/what-did-not-work.md): 48 ideas that did not improve the classifier, with numbers.
- [docs/s1mb-submission.md](docs/s1mb-submission.md): the S1MB English v1 submission, coverage and training-exposure disclosure.

## Status
Selection-level results on open data, mostly single-seed. The 14B model is well below the strongest S1MB entries; see the reports for what was and was not shown. Licence terms for these documents have not been set yet.
