# Tamweel Lite | End-to-End Machine Learning Decision Support

**Course:** SDA-DSC-211 — Advanced Machine Learning Methods  
**Student code:** `211`  
**Environment:** Google Colab (free CPU) | **Data:** Synthetic educational data only

> **Educational use only.** The project simulates a review-prioritization decision. It must not be used to approve or reject real financing applications. This is an independent learner project, not an official SDAIA Academy repository.

## Project overview and business value

Tamweel Lite predicts the probability of a **90-day default outcome** using only information available when a synthetic financing application is submitted. The aim is not simply to maximize predictive accuracy: the model must prioritize cases for **simulated manual review** under a **12% review-capacity limit** and a teaching loss of **10 units per false negative** and **1 unit per false positive**.

The positive class represents approximately **7.9%** of the original training data. Therefore, **Average Precision (AP)**, cost-sensitive loss, calibration, and capacity compliance matter more than accuracy alone.

## From data to a trustworthy decision

```text
Synthetic applications
    ↓
Feature availability and leakage audit
    ↓
Forward, customer-aware validation + honest OOF predictions
    ↓
Compare single models and ensembles (Worth-It Gate)
    ↓
Choose model + OOF threshold under cost/capacity rules
    ↓
Reserved-period sigmoid calibration + threshold transport
    ↓
Frozen inference on unlabeled challenge data
    ↓
12% batch review cap → submission.csv
```

**Leakage control:** The post-outcome variables `days_past_due_60` and `collection_calls` were excluded, leaving **22 eligible predictors**. Preprocessing was learned from training rows. Validation respected chronological order, label maturity, and customer separation. The final comparison used **2,155 outer out-of-fold (OOF) observations across three forward periods**. Challenge labels were neither available nor used.

![Impact of leakage and validation design](artifacts/day2_validation_comparison.png)

*Validation comparison: the near-perfect leaky control is evidence of leakage, not genuine predictive performance.*

## Model comparison and final selection

Six approaches were evaluated using forward OOF evidence:

| Candidate | Mean AP | AP fold SD |
|---|---:|---:|
| LightGBM | 0.34549 | 0.04348 |
| XGBoost | 0.35263 | 0.02904 |
| **Logistic Regression** | **0.39166** | **0.02981** |
| Equal-average ensemble | 0.37170 | 0.03258 |
| Weighted-average ensemble | 0.38942 | 0.02906 |
| Stacking ensemble | 0.38314 | 0.02949 |

![Final single-model and ensemble comparison](artifacts/day5_ensemble_comparison.png)

**Final choice: `KEEP_SINGLE_MODEL` — Logistic Regression.** It had the highest mean AP in the measured outer OOF comparison. None of the ensembles demonstrated enough additional value to pass the predefined **Worth-It Gate**. Fold variation and the limited synthetic validation periods mean the comparison is not a guarantee of future superiority.

## Decision evidence: simulated loss and capacity

The decision rule uses **simulated loss = 10 × FN + 1 × FP**. The raw OOF threshold was approximately **0.16892**. On the final OOF policy evaluation:

| Measure | Result |
|---|---:|
| Outer OOF requests | 2,155 |
| True positives / False positives | 84 / 95 |
| False negatives / True negatives | 151 / 1,845 |
| Recall | 0.35745 |
| Simulated decision loss | **1,111 units** |
| Overall flagged fraction | **11.37%** |
| Review capacity | **12% per period; feasible** |

![Decision and capacity diagnostics](artifacts/day5_policy_regions.png)

*These are OOF development/selection results, not measured performance on the unlabeled challenge set or in real lending.*

## Interpretation and probability reliability

Day 4 used permutation importance and SHAP to inspect a **Day 4 fitted model**. Important features included `bureau_score`, `dti`, and `loan_amount_sar`. The SHAP values were expressed in **raw log-odds**, describing model contributions rather than causal effects. **These Day 4 explanations do not automatically explain the final Day 5 Logistic Regression model.**

![Day 4 SHAP summary](artifacts/shap_beeswarm.png)

On the **Day 4 separate evaluation period**, sigmoid calibration changed Brier score from **0.1130 to 0.0671** and ECE from **0.1469 to 0.0225**. For the **final Day 5 model**, calibration-fit diagnostics were mixed: Brier **0.07647 → 0.07086**, but ECE **0.02112 → 0.03487** and log loss **0.26649 → 0.27730**. The latter are **in-sample calibration-fit diagnostics**, not independent final-model evaluation.

![Final calibration diagnostics](artifacts/day5_calibration_fit.png)

## Frozen challenge delivery

The reserved calibration period contained **836 applications**, including **78 positives**. The final threshold transported to the calibrated probability scale was **0.122258**. The frozen pipeline then scored **2,500 unlabeled challenge applications**:

| Challenge policy result | Count |
|---|---:|
| Applications scored | 2,500 |
| Above transported threshold | 330 |
| Maximum allowed flags (12%) | 300 |
| Final simulated review flags | **300** |
| Threshold-eligible cases removed by capacity | 30 |

![Challenge capacity enforcement](artifacts/day5_challenge_capacity.png)

No challenge outcome labels were available. Therefore **challenge AP, recall, decision loss, and fairness cannot be claimed**.

## How to run and review

1. Open the notebooks in **Google Colab**, using a free CPU runtime and the pinned `requirements-colab.txt` / `constraints.txt` environment.
2. Run [`00_readiness_check.ipynb`](notebooks/00_readiness_check.ipynb), then the five numbered labs in order: [01](notebooks/01_baseline_boosting.ipynb) · [02](notebooks/02_validation_tuning.ipynb) · [03](notebooks/03_cost_sensitive_decision.ipynb) · [04](notebooks/04_explain_calibrate.ipynb) · [05](notebooks/05_final_model.ipynb).
3. Review the executed notebooks, figures, model artifacts, and reports. Use [`99_final_submission_check.ipynb`](notebooks/99_final_submission_check.ipynb) for the course final check on an exact repository snapshot.

**Verified result for the assessed pre-Notebook-99-upload snapshot:** `READY FOR FINAL SUBMISSION`, with notebooks **00–05 passing**. Any subsequent repository changes require a fresh snapshot check to make the same claim about that version.

### Project files

| Resource | Location |
|---|---|
| Model and policy | [`artifacts/final_model/`](artifacts/final_model/) · [`artifacts/final_policy.json`](artifacts/final_policy.json) |
| Model selection and metrics | [`artifacts/ensemble_comparison.csv`](artifacts/ensemble_comparison.csv) · [`artifacts/final_metrics.json`](artifacts/final_metrics.json) |
| Predictions | [`submission.csv`](submission.csv) · [`submission/submission.csv`](submission/submission.csv) |
| Model Card | [`MODEL_CARD.md`](MODEL_CARD.md) |
| Decision Card | [`DECISION_CARD.md`](DECISION_CARD.md) |
| Interpretation report | [`INTERPRETABILITY_REPORT.md`](INTERPRETABILITY_REPORT.md) |
| Ensemble decision | [`ENSEMBLE_DECISION.md`](ENSEMBLE_DECISION.md) |
| Final presentation | [`final_presentation.pdf`](presentation/final_presentation.pdf) |
| Evidence and additional reports | [`evidence/`](evidence/) · [`reports/`](reports/) |

## Repository Structure

The following tree lists the project files by their repository paths.

```text
.
├── artifacts/
│   ├── final_model/
│   │   ├── model.json
│   │   └── model_manifest.json
│   ├── best_params.json
│   ├── calibration_metrics.json
│   ├── cost_curve.png
│   ├── data_check.json
│   ├── day1_comparison_predictions.csv
│   ├── day1_learning_curves.png
│   ├── day1_model_comparison.csv
│   ├── day1_reflection.json
│   ├── day1_roc_pr.png
│   ├── day1_run.json
│   ├── day1_split_membership.csv
│   ├── day2_fold_sizes.png
│   ├── day2_oof_coverage.csv
│   ├── day2_oof_predictions.csv
│   ├── day2_provenance.json
│   ├── day2_reflection.json
│   ├── day2_run.json
│   ├── day2_search.png
│   ├── day2_validation_comparison.png
│   ├── day3_capacity_regions.png
│   ├── day3_cost_sensitivity.csv
│   ├── day3_model_comparison.csv
│   ├── day3_model_report.csv
│   ├── day3_oof_coverage.csv
│   ├── day3_oof_predictions.csv
│   ├── day3_period_capacity.csv
│   ├── day3_provenance.json
│   ├── day3_reflection.json
│   ├── day3_region_audit.csv
│   ├── day3_review_flags.csv
│   ├── day3_roc_pr.png
│   ├── day3_run.json
│   ├── day4_bootstrap.csv
│   ├── day4_capacity.csv
│   ├── day4_local_stability.csv
│   ├── day4_model.txt
│   ├── day4_period_metrics.csv
│   ├── day4_policy_sweep.csv
│   ├── day4_predictions.csv
│   ├── day4_provenance.json
│   ├── day4_reason_codes.csv
│   ├── day4_reflection.json
│   ├── day4_reliability_bins.csv
│   ├── day4_review_flags.csv
│   ├── day4_roles.csv
│   ├── day4_run.json
│   ├── day4_shap_global.csv
│   ├── day4_shap_metadata.json
│   ├── day4_stability_summary.json
│   ├── day5_calibration_fit.png
│   ├── day5_calibration_fit_bins.csv
│   ├── day5_calibration_predictions.csv
│   ├── day5_challenge_capacity.png
│   ├── day5_cost_sensitivity.csv
│   ├── day5_diversity.png
│   ├── day5_ensemble_comparison.png
│   ├── day5_ensemble_gate.json
│   ├── day5_final_provenance.json
│   ├── day5_fold_scores.csv
│   ├── day5_oof_predictions.csv
│   ├── day5_oof_provenance.json
│   ├── day5_period_capacity.csv
│   ├── day5_policy_regions.png
│   ├── day5_probability_correlation.csv
│   ├── day5_project_check.json
│   ├── day5_reflection.json
│   ├── day5_region_audit.csv
│   ├── day5_residual_correlation.csv
│   ├── day5_roles.csv
│   ├── day5_run.json
│   ├── day5_threshold_sweep.csv
│   ├── ensemble_comparison.csv
│   ├── environment.json
│   ├── final_metrics.json
│   ├── final_policy.json
│   ├── fold_audit.csv
│   ├── leakage_audit.csv
│   ├── optuna_results.csv
│   ├── permutation_importance.csv
│   ├── permutation_importance.png
│   ├── readiness_report.json
│   ├── reliability_curve.png
│   ├── review_zone.png
│   ├── runtime_checks.json
│   ├── shap_beeswarm.png
│   ├── shap_values_sample.npz
│   ├── shap_waterfall.png
│   ├── stability_summary.png
│   ├── threshold_metrics.json
│   ├── threshold_sweep.csv
│   ├── validation_report.csv
│   └── validation_summary.csv
├── data/
│   ├── data_contract.json
│   ├── data_manifest.json
│   ├── tamweel_challenge.csv
│   └── tamweel_train.csv
├── evidence/
│   ├── day1/
│   │   ├── day1_comparison_predictions.csv
│   │   ├── day1_learning_curves.png
│   │   ├── day1_model_comparison.csv
│   │   ├── day1_reflection.json
│   │   ├── day1_roc_pr.png
│   │   ├── day1_run.json
│   │   ├── day1_split_membership.csv
│   │   └── environment.json
│   ├── day2/
│   │   ├── best_params.json
│   │   ├── day2_reflection.json
│   │   ├── day2_run.json
│   │   ├── day2_validation_comparison.png
│   │   ├── environment.json
│   │   ├── fold_audit.csv
│   │   ├── leakage_audit.csv
│   │   ├── optuna_results.csv
│   │   └── validation_summary.csv
│   ├── day3/
│   │   ├── cost_curve.png
│   │   ├── day3_oof_predictions.csv
│   │   ├── day3_reflection.json
│   │   ├── day3_region_audit.csv
│   │   ├── day3_run.json
│   │   ├── DECISION_CARD.md
│   │   ├── environment.json
│   │   ├── threshold_metrics.json
│   │   └── threshold_sweep.csv
│   └── day4/
│       ├── calibration_metrics.json
│       ├── day4_bootstrap.csv
│       ├── day4_capacity.csv
│       ├── day4_local_stability.csv
│       ├── day4_model.txt
│       ├── day4_period_metrics.csv
│       ├── day4_policy_sweep.csv
│       ├── day4_predictions.csv
│       ├── day4_provenance.json
│       ├── day4_reason_codes.csv
│       ├── day4_reflection.json
│       ├── day4_reliability_bins.csv
│       ├── day4_review_flags.csv
│       ├── day4_roles.csv
│       ├── day4_run.json
│       ├── day4_shap_global.csv
│       ├── day4_shap_metadata.json
│       ├── day4_stability_summary.json
│       ├── environment.json
│       ├── INTERPRETABILITY_REPORT.md
│       ├── permutation_importance.csv
│       ├── permutation_importance.png
│       ├── reliability_curve.png
│       ├── review_zone.png
│       ├── shap_beeswarm.png
│       ├── shap_values_sample.npz
│       ├── shap_waterfall.png
│       └── stability_summary.png
├── notebooks/
│   ├── 00_readiness_check.ipynb
│   ├── 01_baseline_boosting.ipynb
│   ├── 02_validation_tuning.ipynb
│   ├── 03_cost_sensitive_decision.ipynb
│   ├── 04_explain_calibrate.ipynb
│   ├── 05_final_model.ipynb
│   ├── SDA_DSC_211_Tamweel_Lite.ipynb
│   └── 99_final_submission_check.ipynb
├── presentation/
│   └── final_presentation.pdf
├── reports/
│   ├── DECISION_CARD.md
│   ├── ENSEMBLE_DECISION.md
│   ├── INTERPRETABILITY_REPORT.md
│   └── MODEL_CARD.md
├── scripts/
│   ├── day2_validation.py
│   ├── day3_decision.py
│   ├── day4_trust.py
│   ├── day5_delivery.py
│   ├── day5_final.py
│   ├── inference.py
│   ├── rebuild_final.py
│   └── replay_final.py
├── submission/
│   ├── final_project_manifest.json
│   └── submission.csv
├── tamweel/
│   ├── __init__.py
│   └── inference.py
├── constraints.txt
├── DECISION_CARD.md
├── ENSEMBLE_DECISION.md
├── INTERPRETABILITY_REPORT.md
├── metrics.json
├── SDA_DSC_211_Tamweel_Lite.ipynb
├── MODEL_CARD.md
├── PROJECT_README.md
├── README.md
├── requirements-colab.txt
└── submission.csv
```

**Note:** This tree is based on the final project bundle, with the subsequently uploaded `notebooks/99_final_submission_check.ipynb` included. Files added directly to GitHub after the bundle was generated should be verified against the current repository.

## Limitations and next steps

- **Synthetic data only:** results do not establish real-world creditworthiness, deployment readiness, or regulatory compliance.
- **Limited forward periods:** OOF model comparisons can vary under new populations or time periods.
- **Calibration:** final calibration-fit metrics are mixed and require an independent, labeled future-period check.
- **Interpretation:** SHAP is non-causal; Day 4 explanations cannot be transferred to the final model without a new analysis.
- **Operational and regional review:** monitor drift, labeled AP, calibration, capacity by period, and subgroup error rates with adequate sample sizes before any policy change.

**If new labeled data arrive**, repeat forward/customer-separated evaluation, audit drift and calibration, reassess the Worth-It Gate, and revise the frozen threshold only on a designated development/policy sample—not on the challenge set.

## Course attribution and disclosure

Developed as a learner project for **SDA-DSC-211 — Advanced Machine Learning Methods**, using the course's synthetic Tamweel Lite materials. [SDAIA Academy GitHub](https://github.com/SDAIAAcademy). AI assistance supported learning and documentation; the numeric results reported here come from the executed project evidence. No personal customer data, credentials, or API tokens are required.
