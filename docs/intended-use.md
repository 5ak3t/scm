# Intended use and limits

**Intended use.** Research on typed, non-generative classification: scoring a handful of options for a text in one forward pass, routing and triage experiments, and comparison with other typed-decision models.

**Not for.**
- Decisions that need a guarantee or are safety-, legal-, medical- or financial-critical. The weights ship no calibration, no abstention and no coverage guarantee; the probabilities are an uncalibrated softmax.
- A claim of performance on workflows unlike the training data. Accuracy on unseen workflows is far below in-distribution accuracy (about 60 to 81% on unseen knowledge multiple choice depending on size) and security triage is weak for every size.
- Questions with more than 20 options (v0 does not support them; v1 does, with lower accuracy), or inputs far longer than about 1,000 tokens (training truncation), without your own validation.
- Commercial use without your own review of the training-data licences (see [data and licences](data-and-licences.md): non-commercial and share-alike sources are included).

**Known biases and gaps.** English only. Inherits biases of the Qwen3 base and of the public datasets (toxicity, preference and news sources among them). The training overlap with public benchmarks is described in the [S1MB submission](s1mb-submission.md).
