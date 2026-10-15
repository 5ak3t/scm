# What did not work: 48 ideas that failed to improve our classifier

SCM is a calibrated classifier for typed decisions (pick one of K, yes/no, ordinal score) that answers in one forward pass. This is the list of ideas we tried that did not help, with the numbers we measured. It covers both the large LLM-based model and the small on-device model that came before it.

How to read the numbers: they are development-set results on open data, mostly single-seed, and the small-model ones are noisier (see item 48). "Did not help" means it did not help here, not that it cannot help elsewhere. Each experiment had its accept/reject rule written before the result was known; a few small-model entries were reported by the person who ran them and not re-run.

## Setup shared by the experiments

**Evaluation sets (all open development data; sealed sets are used once and said so).**

| Set | What it is | Size |
|---|---|---|
| `zs_wide` | Unseen-workflow suite from instruction tasks (SNI), service dialogues (SGD), dialogue flows (ABCD) and LLM-synthesised workflows (glmsyn) | 154 groups, 1,842 records |
| `zs_td` | Security-alert triage (expected / contain now / open an investigation / worth watching), deliberately absent from every training pool | 300 records, 1,500 questions, 5 templates; "investigate" is the gold answer for 214 of 300 choice questions (.713) |
| `td` | Typed-decisions workflows that appear in training | 164 records, 820 questions, 15 templates |
| `zs_massive` | 1,000 MASSIVE intents with a label space never seen in training | 1,000 |
| `tickets` | In-house support tickets | 104 records |
| `fresh_sel` | Open half of a fresh unseen-workflow set of knowledge multiple choice (MMLU, MedMCQA and similar, excluded from training by a reserved list); the other half is sealed | 420 choice questions |
| `sec_cve_open` | CVE severity and KEV-listing questions, unseen by any pool | open half only |

**Metrics.** `macro_real` is group-macro accuracy over the wide suite's source families (every group counts equally). NLL is the mean negative log-likelihood of the gold label; ECE uses 15 bins; balanced accuracy averages per-class recall. Prediction sets are conformal (LAC or mass-adaptive scores, Mondrian by question type), calibrated for 90% coverage with workflow-level thresholds (group quantile level beta .2 to .3); "coverage" is the share of questions whose gold label is in the set, and "set size" is the mean number of options returned. Certified automation (CA) is the share of questions answered automatically at a stated risk (.10 or .15) with a Clopper-Pearson bound at delta .10 on one half of each suite, measured on the other.

**Statistics and accept/reject rules.** Intervals are paired or stratified-bootstrap 95% intervals (t intervals for group-macro scores), clustered by record. Rules were fixed before each result: for the small model, a change was kept only with dev NLL down by at least .010 and an interval below zero (or an equal metric at half the size or latency); for students, macro_real up by at least +.05 with a positive lower bound and no other suite down by more than .02; for the large joint models, fresh-set accuracy at least .628, dev total at least .7091, and security-triage accuracy at least .6513. Dev numbers select; they are not results.

**Models and recipes.**
- Small model (`ej`): `e5-small-v2` encoder trimmed to 11,680 word pieces, 2-bit weights and 3-bit attention (quantisation-aware training), eleven small expert heads combined by a log-linear pool per calibration key, Bayesian hierarchical stacking for unseen option slots, and a cross-fitted selective head; about 11 to 30 MB depending on the variant, no LLM at inference.
- Teacher: Qwen3-Reranker-8B, QLoRA (nf4), listwise softmax over options, yes/no questions as a statement plus its negation, state tail truncated to 1,024 tokens, early stopping on held-out pool groups, pool v3 (58,091 records, 72,147 questions: 68,714 train and 3,433 held out).
- Students: Qwen3-Reranker-0.6B (full fine-tune, AdamW 2e-5) and 4B (LoRA r16, alpha 32), each on gold plus teacher soft labels.
- Joint-option models: Qwen3-Base (0.6B to 14B) with LoRA r16, alpha 32, all attention and MLP projections, one epoch of 3,733 steps at 16 questions per step, prompts left-truncated at 1,024 tokens, random option order in training. Learning rate 1e-4 in fp16 for 4B and smaller; 5e-5 in bf16 with a non-finite-gradient guard for 8B and 14B. Scoring uses the next-token logit of the option letter, averaged over 2 to 4 option orders.
- Pools: v3 (12 groups, 58,091 records), v4 (25 groups, 76,414), v5 (168 groups, 124,334; capped at 500 per group).

| Theme | Items |
|---|---|
| Training the large model | 1 to 6 |
| Data | 7 to 12 |
| Priors and debiasing | 13 to 18 |
| Calibration and guarantees | 19 to 26 |
| Model structure | 27 to 36 |
| Size and speed | 37 to 46 |
| Prompting a hosted LLM | 47 |
| Our own yardstick | 48 |

## Training the large model

**1. Label-free noise-floor correction (batch calibration).** Subtract each template's constant answer-key preference, estimated from the other unlabeled records of that template. On Qwen3-Reranker-8B it scored **-.088 [-.133, -.042]** macro on the wide zero-shot suite, -.149 on security triage and -.068 on typed decisions, and failed all three adoption clauses. The correction also removes the true label prior; in our suites the most frequent key covers .54 to .67 of a template, so it flipped confident, correct answers.

**2. Frozen 8B features plus a trained head.** An MLP head scored **-.052 [-.129, +.025]** macro against the zero-shot 8B and -.088 on security triage; it overfit after epoch 1 (train loss .44 to .36 while held-out loss rose). The linear head never beat zero-shot on held-out groups. The signal is not linearly available in the last layer; LoRA fine-tuning is what moved the numbers.

**3. Distilling an 8B teacher into a 0.6B student.** Soft labels from a fine-tuned 8B teacher, 0.5 CE + 0.5 KL, three seeds, full fine-tune. Macro **-.045 [-.172, +.081]** against the student's own zero-shot (the rule needed +.05), security triage **-.106**, seed spread .392 to .512, and .262 below the teacher. The 8B teacher trained on the same pool did not lose robustness (+.044). A 4B LoRA student passed; later 0.6B and 1.7B runs also fail (.505 and .605 on unseen knowledge multiple choice).

**4. Averaging two scoring rules.** Accuracy .7291 to .7308, NLL .634 to .634. Only yes/no differs between rules, and averaging moved it both ways (.728 to .767 on one suite; .733 to .695, .817 to .794 and .827 to .817 on three others). For the per-option model, permuting option order cannot change a score, so order averaging only matters for the joint-option format.

**5. fp16 and unguarded bf16 training of 8B and 14B.** Non-finite gradients, NaN by step 40, one run degrading mid-training. A non-finite-gradient guard fixed it (58 of 3,733 steps skipped); a recipe fix, not a modelling result.

**6. Joint-option letters as a replacement for per-option scoring.** +.17 on unseen multiple choice, but security triage fell to .47 to .58 against .70. No single joint student met both rules; the passing ensemble was assembled after seeing the failure, so its numbers carry a selection effect.

## Data

**7. Many more workflow groups.** Accuracy on unseen knowledge multiple choice as the pool grew from 12 to 25 to 168 groups: **.733, .714, .719**. Model size moved it instead: .719 (4B), .764 (8B), .810 (14B).

**8. Broad text, dialogue and instruction data for the small model.** Adding instruction tasks, dialogue flows and service dialogues (215 to 300 groups) cut security-triage accuracy from .466 to .359 and .417 and raised NLL on a never-seen-intent suite from .48 to between .57 and .63. Generic diversity pulled the model off the typed-decision format.

**9. Structured tabular workflows.** Adding 36 UCI tabular workflows (44 groups) was significantly worse: security-triage accuracy .385 to .263, NLL 1.164 to 1.219, intent-suite NLL .449 to .491. Tabular groups shifted the hierarchical stacking away from text workflows.

**10. LLM-synthesised workflows in the decision style.** Adding 85 synthetic workflows lowered security-triage NLL (1.127 to 1.086) but accuracy still fell (.466 to .436), and adding a preference-data set (HelpSteer2) on top gave the best overall dev metric of that series with the worst triage accuracy (.380).

**11. Doubling the ticket data.** 631 to 1,231 tickets moved dev NLL from .8168 to .8163. The plateau was not a data-volume limit.

**12. Soft labels from a small LLM.** NLL .8394 and .8930 against .8169. The teacher agreed with the intended labels only 81%, 70% and 55% of the time on choice, yes/no and score, so its soft labels hurt the last two.

## Priors and debiasing

**13. A better label prior for the triage failure.** The best possible per-option offset (fitted on gold labels, in-sample and 5-fold CV) gives accuracy .72 to .74 but **balanced accuracy only .32 to .34 for every model**; it turns each model into "always investigate". Plain accuracy on that suite is a weak yardstick (the majority answer scores .713); use balanced accuracy.

**14. Label-free per-workflow prior correction.** Macro +.0141 [+.0026, +.0269], below the +.02 bar. Shipped as experimental, not as a default.

**15. Online test-time label shift.** It looked like a -.032 NLL gain on one suite, then an audit showed it exploited how distractor options were sampled (option-text frequency alone gave .633 against .240 chance). Rejected as leakage; on fixed schemas the method hurt.

**16. Content-free (null-state) calibration.** A content-free "N/A" null prediction hurt (NLL 0.7855 against about 0.74). The schema-only null (keys kept, values blanked) mixed into the pool reached only -.0082, short of the -.010 bar.

**17. Product-of-experts debiasing.** Dividing out a bias-only expert raised security-triage accuracy from .383 to .473, but the dev metric improved only -.0088, short of the -.010 bar.

**18. Combining 16 and 17.** Not additive: the pooling mode for unseen JSON flipped and lost the triage gain (-.0023).

## Calibration and guarantees

**19. Static conformal calibration on new workflows.** Coverage on a sealed fresh set: shipped fit **.705**, repaired fit **.841** against a .88 target (medmcqa .778 against .85). A conservative variant reached **.9015 [.870, .930]** only by returning **3.2 options per 4-option question**. Coverage is bought with set size; accuracy was unchanged (.553).

**20. A confidence-only shift detector for the threshold.** Coverage .864, .860 and .874 against the static rule's .862. Only a withhold rule (flag workflows below the 5th percentile of calibration confidence and return the full option set) helped: .862 to .890 at +0.06 options. Overconfidence differs by family (ECE .020 on calibration data, .077 on the open half), so confidence alone cannot extrapolate.

**21. Risk-controlled abstention.** Refitting on the larger calibration set removed the guarantee: choice had no certifiable threshold (the older tau .85 accepted answers at error .075 against a .05 budget), and yes/no reached only .922 out-of-fold selective accuracy against the .95 needed. We ship prediction sets with no abstain flag.

**22. Fully group-honest calibration.** Flat (-.0021). Finding: the option-text tilt each new workflow brings cannot be removed by calibration.

**23. Group-honest nested cross-fitting.** The leakage-free property held (160 checks, 0 violations), but the metric got worse by +.026, three-quarters of it from security triage (NLL 1.131 to 1.212, worse than uniform at 1.153). The earlier best model's triage edge rested on a leaky trust estimate.

**24. Stacked mixtures and shrunk selective heads.** A per-cell stacked mixture overfit with only 3 workflows in the JSON regime (+.0037 worse); a conservative selective head cut worst-group harm (+.059 to +.015) but raised security-triage ECE from .050 to .133, so it was left off. Three workflows cannot support transferable pool maths.

**25. A learned confidence and an ordinal risk.** Stacking pool, native and LLM confidences as the abstention score, and an ordinal-aware risk for score questions, were both discarded on prototype evidence.

**26. Pooling a hosted LLM with the small model.** Gains seen on a 104-record dev set mostly did not replicate on test (best test NLL .679 to .642): regression to the mean.

## Model structure

**27. A slot-free relational reader** (value-free fact pairs read by attention): equal (-.0010).

**28. A single-pass multi-option reader** (GLiClass/UniMC style on the frozen encoder): +.0091 worse, worse than the deep expert on 7 of 8 pool groups, security-triage NLL 1.131 to 1.171.

**29. Set attention over a question's options.** Plain-text only: -.0086 on the wide suite with a confidence interval below zero (accuracy .3644 to .3557); the label-agnostic variant gained +.0165 on the metric but not on the wide suite (+.006 [-.014, +.031]).

**30. A large-teacher NLI expert as a second pool expert.** Flat (-.0015); the swap-only variant (.7407) was worse on the intent suite.

**31. An NLI expert that helped but could not ship.** Dev improved from .7310 to .7555, but the checkpoint was trained on a non-commercial NLI dataset, which conflicts with releasing open weights.

**32. Multi-teacher fine-tuning of the query tower.** +.0307 worse (security triage NLL 1.131 to 1.215).

**33. Distilling a reranker into a rich scorer** (Margin-MSE on pool texts): +.0305 worse.

**34. Encoder-level distillation of a reranker into the low-bit encoder.** Held-out Margin-MSE fell 13.6% against a 20% bar; stopped.

**35. Decoupled KD (gold CE + TCKD + NCKD) with calibrated two-teacher targets.** In-fit held-out loss improved (.653 to .577) but the system was flat (+.0018). Strictly honest leave-one-group-out targets cost .02 to .03.

**36. Evidence-weighted (pseudo-BMA) pool hierarchy.** Flat to worse on dev (+.002).

## Size and speed

**37. 2-bit and 3-bit encoder with an int8 head.** 11.3 MB instead of 18 to 29 MB, but heads fit on 4-bit features lost +0.040.

**38. A shared encoder trunk** (one lower pass feeds all readers): about 2x faster (0.54x and later 0.39x compute), but the blurred state vector cost accuracy (typed decisions NLL .753 to .798; later security-triage NLL 1.122 to 1.234).

**39. A shallower encoder (exit at layer 8).** Cold latency 139.8 to 78.6 ms (0.56x, 0.50x was needed) with the dev metric flat to worse (.7357 to .7401; intent-suite NLL .497).

**40. A two-stage cascade with a static first stage** (11.3 MB, deferring to the full model): +.0058 worse, and a cascade alone cannot reach the latency target.

**41. Encode each state once.** 25 to 54% fewer tokens and cold 61.9 ms, but +0.026 worse even after a de-contextualisation fix (+0.020).

**42. Certified early exit at layer 6.** The label-free bound held on dev (realised KL .00085 against .002), but only one suite accepted 10% or more (13.6%) and the speed-up was 1.03x. Exit features were cosine .93 to .96 where the heads need above .99.

**43. A trained exit block.** Closer features (cosine .978 and .985) but slower (91.2 ms against 86.3).

**44. Label-free certificates across new groups.** A certified pre-exit score passed on the pool but violated its budget on dev (KL per accepted question .012 to .061 against .002). Label-free scores do not transfer to new groups.

**45. A group-level certificate.** The theorem needs about 22 groups for a delta of .10 (we had 8): zero violations but about 0% acceptance.

**46. Faster QLoRA and CPU serving.** Unsloth was never run (every release that fits our stack imports AGPL-3.0 modules). Serving the 0.6B reranker on CPU through OpenVINO and ONNX Runtime failed its parity and latency gate.

## Prompting a hosted LLM

**47. Few-shot examples, base rates in the prompt, thinking mode, one request per question.** Dev gains mostly did not replicate (item 26); per-question requests were discarded; thinking cost about 16 s per record against 2.6 s.

## Our own yardstick

**48. Fit-to-fit noise.** The same code on the same pool, fitted twice with only the pipeline changed, moved security-triage accuracy from .466 to .419 and NLL from 1.127 to 1.147. A single held-out workflow therefore has a noise floor around .05 accuracy; some of the "gains" we chased were at that level.

## What the failures have in common

Most failures came from one assumption that held on the training workflows and broke on a new one: label balance within a template (items 1, 13, 15 and 21), calibration data resembling the target (19, 20, 22 to 24), more or broader data meaning more transfer (7 to 10), a smaller or cheaper model keeping what a larger one keeps (3, 37 to 45). Model size was the only lever that reliably moved unseen-workflow accuracy.
