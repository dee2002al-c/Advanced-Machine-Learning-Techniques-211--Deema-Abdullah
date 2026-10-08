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

## Limitations and next steps

- **Synthetic data only:** results do not establish real-world creditworthiness, deployment readiness, or regulatory compliance.
- **Limited forward periods:** OOF model comparisons can vary under new populations or time periods.
- **Calibration:** final calibration-fit metrics are mixed and require an independent, labeled future-period check.
- **Interpretation:** SHAP is non-causal; Day 4 explanations cannot be transferred to the final model without a new analysis.
- **Operational and regional review:** monitor drift, labeled AP, calibration, capacity by period, and subgroup error rates with adequate sample sizes before any policy change.

**If new labeled data arrive**, repeat forward/customer-separated evaluation, audit drift and calibration, reassess the Worth-It Gate, and revise the frozen threshold only on a designated development/policy sample—not on the challenge set.

## Course attribution and disclosure

Developed as a learner project for **SDA-DSC-211 — Advanced Machine Learning Methods**, using the course's synthetic Tamweel Lite materials. [SDAIA Academy GitHub](https://github.com/SDAIAAcademy). AI assistance supported learning and documentation; the numeric results reported here come from the executed project evidence. No personal customer data, credentials, or API tokens are required.
