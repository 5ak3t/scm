# SCM-max 14B v0 on S1MB English v1

Submission: https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/37 (results dataset pull request, one model folder `5ak3t__scm_max_14b_v0`, 137 result files).

Model: https://huggingface.co/5ak3t/scm-max-14b-v0 at revision `1e983c1a1a386b9a49f04fa219e5b1ff8cd12058` (LoRA on `Qwen/Qwen3-14B-Base` @ `0b0bd3732e2c374d483664439ea334928b65f304`; adapter SHA-256 `2c3f9a495c544ba0ca07f03c1364c28805de8cfe574aeec715af4cf4ca3764ed`).

## Configuration (fixed before any S1MB score)
fp16 base with the adapter merged, sdpa attention, four option orders averaged, temperature 1, no calibration, input limit 16,384 tokens with overflow rejected, at most 20 candidates per question, 16 cases per call. One H100, 3,589 s. No S1MB label or score was used to choose any setting.

## Coverage
131 of 137 benchmarks complete. Six (banking77, clinc, gretel-pii, hwu64, massive, mtop) contain questions with more than 20 candidates; the lettered-option format cannot hold them and the adapter rejects them instead of splitting, so all 651 cases (1,104 decisions) are saved as failures. Nothing else failed. The submission is therefore partial and does not qualify for aggregates that need complete coverage.

## Scores (baseline-adjusted, 0 to 100, equal benchmark weights)
| | Noul (59) | Choice (51 complete of 57) | Score (21) | gen-noul | gen-choice | gen-score |
|---|---|---|---|---|---|---|
| SCM-max 14B v0 | 50.09 | 52.9 | 32.31 | 88.00 | 96.71 | 74.07 |
| Jev 1.13 (pinned reference, same benchmarks) | 64.63 | 65.2 | 46.92 | 99.00 | 98.68 | 91.14 |

The shared leaderboard's choice aggregate is N/A because of the six incomplete benchmarks; the choice figures above are our means over the 51 complete ones. SCM-max is well below the reference. The training-exposure notes below apply to the three typed-decisions benchmarks in particular.

## Training exposure (submitter's audit, to the best of our knowledge)
- Bekko System One and Saihai data were not used. No S1MB test input, label or score was used for training, prompts, option orders, limits or checkpoint selection.
- Input-only overlap audit (normalised exact match, or at least half of the 8-word shingles present in the training pool) against the English test inputs: 101 cases overlap. 30 of 100 inputs in each of `laya--typed-decisions-{choice,noul,score}-test-v1` also occur in training (the pool contains the `LocalLLaMA/typed-decisions` train split), declared as `evaluation_examples`. Targets were not compared. Single-case overlaps occur in banking77, clinc, dbpedia, few-nerd, hh-rlhf, nanodbpedia and ud-ewt.
- Declared `same_source_family`: banking77, clinc, dbpedia, go_emotions, AG News, Enron spam, PAWS, OpenBookQA, AQuA-RAT, Civil Comments, HH-RLHF, ContractNLI (via LegalBench). Related domains (NLI, moderation, sentiment, legal, spam) are not enumerated, and the Qwen3 pretraining data is not audited.
- The training pool includes 780 synthetic support tickets written by an LLM from deterministic specifications; no other LLM-generated labels were used.

The audit report is `s1mb-overlap-report.json` in the model repository.

---

# Version 1 on S1MB English v1

Models: https://huggingface.co/5ak3t/scm-max-14b-v1 (measured revision `b0b58112d57581a86d74054c4c604251a8a7647d`) and https://huggingface.co/5ak3t/scm-mid-4b-v1 (measured revision `39c9a74e96dca1bafd7c3e0b938d3b6476a2db97`). The cards were rewritten after measurement; the adapter weights did not change. Draft results pull request for the 14B model: https://huggingface.co/datasets/hotchpotch/s1mb-result/discussions/38 (not yet opened; the 4B pull request is not created yet).

Configuration (fixed before any v1 S1MB score): fp16 base with the adapter merged, four option orders averaged, temperature 1, no calibration, input limit 16,384 tokens with overflow rejected, 16 cases per call, adapter renderer `scm-joint-letters-v2` (up to 255 labels, nothing split or truncated). Each model was measured in three parallel runs of one configuration on one H100 each, 137 of 137 benchmarks complete, zero failed decisions and zero abstentions. The v0 measurements stay unchanged in the earlier pull request.

## Training exposure (v1)
This replaces the v0 statement that Bekko System One data was not used: **v1 did use it** (training splits only, revision 9c673a09, 65 families). The submission declares 97 `same_source_family` entries. Inputs matching an S1MB English test input exactly or by 8-word overlap were removed (4,524 candidates). A record-level audit of the final pool still finds **304 test cases with an approximate match** out of 15,609; they are approximate-match candidates, not confirmed uses of evaluation inputs, and the affected entries say so in their notes. The retained training rows are not republished (access recorded as private); what we cannot establish is stated as unknown. v1 also differs from v0 in training on questions with more than 20 options.
