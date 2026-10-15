# Changelog

## v1 (2026-10-15)
- New: `scm-max-14b-v1` and `scm-mid-4b-v1` (LoRA on Qwen3-14B-Base and Qwen3-4B-Base), trained on pool v6 lite including questions with up to 160 options; prompt format supports up to 255 options.
- `scm-min-1.7b-v1` was trained but did not pass the release check (held-out accuracy below v0); not released.
- S1MB English v1: 14B v1 scores 55.46 task average (v0 45.92) with all 137 benchmarks complete; 4B v1 30.15. Training on the source families' training splits is disclosed (97 entries, 304 approximate-match test cases).
- New documents: [what v1 changed](docs/v1.md); size and release-check figures.
- 79 readout experiments (thresholds, medians, shrinkage, LoRA scale, option orders, empty-input calibration) did not pass the pre-registered rule; v1 keeps the plain readout.
- v0 releases are unchanged and still available.

## v0 (2026-10-15)
- First public release: three LoRA adapters (`scm-min-1.7b-v0`, `scm-mid-4b-v0`, `scm-max-14b-v0`) on Qwen3 base models.
- S1MB English v1 evaluation of `scm-max-14b-v0` submitted (131 of 137 benchmarks complete).
- Headline: rated typed decisions; positioning against Jev and the open models; positioning in the README.
- Documents: results, usage, data and licences, intended use, S1MB submission and overlap report.
