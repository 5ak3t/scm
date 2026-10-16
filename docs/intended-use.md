# Intended use and limits

**Intended use.** Research on typed, non-generative classification: scoring a handful of options for a text in one forward pass, routing and triage experiments, and comparison with other typed-decision models.

**Not for.**
- Decisions that need a guarantee or are safety-, legal-, medical- or financial-critical. The weights ship no calibration, no abstention and no coverage guarantee; the probabilities are an uncalibrated softmax.
- A claim of performance on workflows unlike the training data. We have not measured v1 on unseen workflows beyond the held-out groups described in [how v1 was built and checked](v1.md), which come from families it also trained on; expect lower accuracy on a genuinely new workflow and test on your own data first.
- Questions with many options (up to 255 are supported, with lower accuracy on very wide ones), or inputs far longer than about 3,000 tokens (training truncation), without your own validation.
- Commercial use without your own review of the training-data licences (see [data and licences](data-and-licences.md): non-commercial and share-alike sources are included).

**Known biases and gaps.** English only. Inherits biases of the Qwen3 base and of the public datasets (toxicity, preference and news sources among them). The training overlap with public benchmarks is described in the [S1MB submission](s1mb-submission.md).
