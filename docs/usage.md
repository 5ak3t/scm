# How to use the models

SCM adapters are LoRA weights for `Qwen/Qwen3-1.7B-Base`, `Qwen3-4B-Base` and `Qwen3-14B-Base`. Each adapter's `adapter_config.json` pins the base model and its exact revision. Load the base in fp16, apply the adapter, and (optionally) merge it. There is no generation step: you run one forward pass per prompt and read one next-token logit per option.

## The prompt
Every question is shown with **all** of its options as lettered lines. The prompt is exactly:

```
The following is a multiple choice question. Choose the single best answer.

<instructions>

Question: <state>
A. <option text>
B. <option text>
C. <option text>
Answer:
```

- `<instructions>` is the question text (a system prompt first, if you have one). `<state>` is the text to decide about; a JSON state is inserted as compact JSON text.
- Options are the option descriptions, in whatever order you choose to show them. Use the labels A to Z in order, then the two-letter labels AA, AB, ... for questions with more than 26 options (the exact label rule is in `l87_joint_prompt.py` in each model repository).
- Question types: **choice** (your K candidates), **noul** (two options, the "false" meaning first and the "true" meaning second, each described in words), **score** (the ordered level descriptions, lowest first).

## Scoring
1. Tokenise the prompt and take the logits at the last position.
2. For the letters you showed, read the logit of the token for a space followed by the letter (" A", " B", ...). These are the option scores.
3. Take the log-softmax over just those letters.
4. **Average over several option orders.** Score the question under four orders (the original order, the reversed order, and two cyclic shifts of the original; for two options just the original and the reversed). For each order, map the letter scores back to the original options, then average the four log-softmax vectors per option.
5. Apply a softmax to the averaged vector (temperature 1). The result is one probability per option. For a score question, the expected score is the sum of probability times the numeric level.

Averaging orders removes most of the letter-position bias; one order alone is usable but less accurate.

## Limits
- **Score questions:** each option is shown by its description, in level order, and the model returns one probability per option; the numeric value of an option is applied by whoever reads the result (for example an evaluator that takes the expected value), not shown by the adapter. Many Score questions already write the value in the description (for example "3.0: a direct answer"); where they do not, the descriptions are ordered from lowest to highest. Keep your levels evenly spaced and in ascending order, and put the numbers in the descriptions if the scale matters.
- **Option limit:** up to 255 options per question (labels A to Z, then single-token two-letter labels). Accuracy on very wide questions is lower than on short ones.
- The models were trained on prompts truncated to 3,072 tokens. Longer prompts work mechanically (the S1MB runs allowed up to 16,384 tokens) but are outside the training distribution and are not expected to be as accurate.
- The released probabilities are the raw softmax. No temperature, calibration or abstention is applied, and no coverage guarantee comes with these weights.

## Choosing a size
`scm-max-14b-v1` is the most accurate by a clear margin (S1MB task average 55.5 against 30.2); `scm-mid-4b-v1` is a quarter of the cost. There is no 1.7B release. See [results](results.md).
