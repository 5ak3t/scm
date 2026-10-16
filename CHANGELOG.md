# Changelog

## v1 (2026-10-15)
- Models: `scm-max-14b-v1` and `scm-mid-4b-v1` (LoRA on Qwen3-14B-Base and Qwen3-4B-Base), trained on pool v6 lite including questions with up to 160 options; the prompt format supports up to 255 options.
- A 1.7B v1 was trained but did not pass the release check; not released.
- S1MB English v1: 14B scores 55.46 task average and 4B scores 30.15, both with all 137 benchmarks complete. Training on the source families' training splits is disclosed (79 declared entries, 304 approximate-match test cases).
- Documents: [how v1 was built and checked](docs/v1.md), [training data and licences](docs/data-and-licences.md), results, usage, intended use and the S1MB submission.
- 79 readout experiments (thresholds, medians, shrinkage, LoRA scale, option orders, empty-input calibration) did not pass the pre-registered rule; v1 keeps the plain readout.
- The S1MB adapter source used for the submitted runs is published in each model repository, and the Score-option behaviour is documented in [usage](docs/usage.md).
