# Advanced-Machine-Learning-Techniques-211--Deema-Abdullah
# Tamweel Lite — Advanced Machine Learning Methods

## Project Overview

Tamweel Lite is a synthetic educational machine learning project developed progressively across five labs. The project studies a binary classification problem in which the objective is to estimate whether a synthetic loan applicant will experience a default event within 90 days of application.

Only information available at application time is used for modeling. The project focuses not only on predictive performance, but also on leakage prevention, time-aware validation, cost-sensitive decision making, calibration, interpretability, capacity constraints, and reproducibility.

> **Important:** This project uses synthetic data for educational purposes only. It must not be used for real lending, financing, credit, or customer decisions.

---

## Project Objectives

The project was developed progressively across five labs:

1. Establish baseline models and compare boosting methods.
2. Detect leakage and implement honest time-aware validation.
3. Introduce cost-sensitive decision making and operational capacity constraints.
4. Examine model interpretation, calibration, and stability.
5. Compare single models and ensemble approaches, freeze the final procedure, and generate reproducible challenge predictions.

The final workflow preserves time ordering, target maturity, and customer separation where required.

---

## Repository Structure

```text
.
├── artifacts/
│   ├── final_model/
│   ├── day1_...
│   ├── day2_...
│   ├── day3_...
│   ├── day4_...
│   ├── day5_...
│   ├── ensemble_comparison.csv
│   ├── final_metrics.json
│   └── final_policy.json
│
├── notebooks/
│   ├── 01_baseline_boosting.ipynb
│   ├── 02_validation_tuning.ipynb
│   ├── 03_cost_sensitive_decision.ipynb
│   ├── 04_explain_calibrate.ipynb
│   └── 05_final_model.ipynb
│
├── reports/
│   ├── DECISION_CARD.md
│   ├── INTERPRETABILITY_REPORT.md
│   ├── MODEL_CARD.md
│   └── ENSEMBLE_DECISION.md
│
├── submission/
│   └── submission.csv
│
├── presentation/
│   └── final_presentation.pdf
│
└── README.md
Lab 1 — Baseline Boosting
Objective
The first lab established a baseline modeling workflow and compared Logistic Regression with XGBoost and LightGBM.
The training data contained approximately 7.9% positive cases, making the problem imbalanced.
Model Comparison
Model	ROC-AUC	Average Precision
Logistic Regression	0.8213	0.3258
XGBoost	0.8124	0.3338
LightGBM	0.8138	0.3248


Logistic Regression provided competitive performance while remaining simple and fast.
Average Precision was considered especially useful because the positive class is relatively rare and the project is concerned with identifying positive cases rather than relying on accuracy alone.
Initial Limitation
The initial random split was useful as a baseline but did not fully represent the temporal and repeated-customer structure of the data. This limitation motivated the more rigorous validation strategy introduced in Lab 2.
Lab 2 — Validation and Tuning
Leakage Audit
A feature availability audit identified two post-outcome variables that should not be used as predictors:
- days_past_due_60
- collection_calls
These variables were removed because they become available after the application decision point.
A total of 22 eligible predictors remained.
Honest Validation
Three forward validation periods were constructed while respecting:
- chronological ordering,
- 90-day target maturity,
- customer separation,
- training-only preprocessing.
The honest validation design produced zero shared customers between training and validation within the evaluated folds.
Leakage Effect
The comparison clearly demonstrated the effect of leakage:
Validation Scheme	Mean ROC-AUC	Mean AP
Leaky random control	0.9999	0.9988
Clean random control	0.8010	0.3110
Clean time/group fixed	0.7976	0.3153
Clean time/group tuned	0.7855	0.3133


The extremely high performance of the leaky control was not realistic model quality. It demonstrated how post-outcome information can artificially inflate validation metrics.
The honest OOF evaluation covered 5,039 eligible observations.
Lab 3 — Cost-Sensitive Decision Making
Business Cost Assumption
The educational decision policy assigned:
- False Negative cost = 10 units
- False Positive cost = 1 unit
A maximum review capacity of 12% was also introduced.
This means model quality alone was not sufficient. The selected decision threshold also needed to respect the operational capacity constraint.
Strategy Comparison
Three approaches were evaluated:
- Unweighted
- Weighted
- Oversampled
The pooled Average Precision values were approximately:
Strategy	Pooled AP
Unweighted	0.3091
Weighted	0.3100
Oversampled	0.3159


The weighted strategy was retained according to the predefined decision procedure.
Decision Threshold
The selected threshold was approximately:
0.6583
At this threshold:
- flagged requests: 526
- recall: approximately 0.409
- educational loss: 2,639 units
- capacity constraint: feasible
Capacity was also checked separately within each validation period rather than only across the pooled data.
Regional Diagnostic
Regional false-positive rates were examined descriptively.
The observed differences were small, with an approximate maximum gap of 0.65 percentage points in the evaluated data.
These values are descriptive diagnostics only and should not be interpreted as a fairness certification or statistical significance test.
Lab 4 — Explainability and Calibration
Data Roles
Separate data roles were created for:
- model fitting,
- calibration,
- policy selection,
- evaluation.
This separation reduced the risk of using the same observations for incompatible modeling decisions.
Feature Importance
Permutation importance identified bureau_score and dti as the strongest features in the evaluated model.
Other influential variables included:
- loan_amount_sar
- savings_balance_sar
- prior_defaults
- recent_inquiries
- income_sar
- utilization_ratio
SHAP Analysis
SHAP was used to examine global and local model behavior.
The strongest global SHAP features included:
1. bureau_score
2. dti
3. loan_amount_sar
4. savings_balance_sar
5. existing_obligations_sar
SHAP values in this analysis explain the raw model output in log-odds rather than calibrated probabilities.
The Day 4 explanation evidence applies to the Day 4 model and should not automatically be attributed to the final Day 5 Logistic Regression model.
Calibration
On the Day 4 evaluation period:
Metric	Raw	Sigmoid
ROC-AUC	0.7708	0.7708
Average Precision	0.2587	0.2587
Brier Score	0.1130	0.0671
ECE	0.1469	0.0225


Ranking performance remained unchanged, while probability-quality diagnostics improved in this evaluation period.
These results do not prove perfect calibration or future performance.
Lab 5 — Final Model and Ensemble Decision
Nested Forward OOF Evaluation
The final model-selection procedure used nested forward out-of-fold validation.
The outer OOF evaluation contained:
2,155 requests across three forward validation periods.
Inner OOF predictions were used to learn ensemble components, while outer OOF predictions were used for model comparison.
The procedure maintained time ordering, label maturity, and customer separation.
Ensemble Comparison
Three single models and three ensemble approaches were compared:
- LightGBM
- XGBoost
- Logistic Regression
- Equal averaging
- Weighted averaging
- Stacking
Model	Mean AP	Fold SD	Mean Brier	Mean ECE
LightGBM	0.34549	0.04348	0.06608	0.02311
XGBoost	0.35263	0.02904	0.06566	0.02277
Logistic Regression	0.39166	0.02981	0.06327	0.01882
Equal Average	0.37170	0.03258	0.06435	0.02038
Weighted Average	0.38942	0.02906	0.06332	0.01772
Stacking	0.38314	0.02949	0.06603	0.03106


None of the ensemble approaches produced enough improvement over the best single model to pass the predefined worth-it gate.
Final Model Decision
KEEP SINGLE — Logistic Regression
Logistic Regression achieved the highest mean Average Precision:
Mean AP = 0.39166
The weighted ensemble came close, but its mean AP was slightly lower at 0.38942.
Therefore, the additional ensemble complexity was not justified by the measured OOF evidence.
Final Decision Policy
The raw OOF decision threshold was approximately:
0.16892
The OOF policy produced an overall flag fraction of approximately:
11.37%
This remained within the 12% operational capacity constraint.
The decision policy used the educational cost structure:
FN cost = 10
FP cost = 1
Regional results were examined as descriptive diagnostics only and are not evidence of certified fairness.
Final Calibration
The selected Logistic Regression procedure was refitted and a sigmoid calibration mapping was learned using the reserved calibration period.
The calibration set contained:
- 836 requests
- 78 positive cases
The final transported threshold on the calibrated probability scale was approximately:
0.122258
The calibration metrics reported during this stage are fit diagnostics on the reserved calibration sample. They should not be interpreted as independent challenge performance.
Challenge Predictions
The frozen model and decision policy were applied to all:
2,500
unlabeled challenge requests.
Before applying the full-batch capacity limit:
330
requests exceeded the transported threshold.
The 12% capacity limit allowed:
300
final review flags.
Therefore:
Item	Result
Challenge requests	2,500
Threshold-eligible requests	330
Final capacity	300
Final review flags	300
Removed by capacity	30
Capacity satisfied	Yes


The challenge dataset does not contain outcome labels. Therefore, challenge AP, ROC-AUC, loss, FPR, or other supervised performance metrics are not reported.
A decision of 1 represents a simulated educational review flag. It does not represent a real financing rejection or approval decision.
Final Results Summary
Component	Final Result
Final model	Logistic Regression
Ensemble decision	KEEP SINGLE
Final selection mean AP	0.39166
Outer OOF observations	2,155
OOF raw threshold	~0.16892
Calibrated transported threshold	~0.122258
Capacity limit	12%
Challenge requests	2,500
Threshold eligible	330
Final review flags	300
Challenge labels used	No


Reproducibility
The project was executed using a controlled environment with:
- Python 3.13.16
- NumPy 2.1.3
- pandas 2.2.3
- scikit-learn 1.6.1
- XGBoost 3.4.1
- LightGBM 4.6.0
- SHAP 0.52.0
- Optuna 4.5.0
- random seed = 211
- CPU execution
- n_jobs = 2
The final exported model is stored under:
artifacts/final_model/
The frozen decision policy is stored in:
artifacts/final_policy.json
The final metrics and supporting evidence are stored in:
artifacts/final_metrics.json
and the other Day 5 artifact files.
The final challenge predictions are stored in:
submission/submission.csv
The exported model passed the replay check, confirming that the saved model reproduced the exported challenge predictions.
Reports
The repository contains four main project reports:
- reports/DECISION_CARD.md
- reports/INTERPRETABILITY_REPORT.md
- reports/MODEL_CARD.md
- reports/ENSEMBLE_DECISION.md
These reports document the decision policy, interpretability evidence, final model, ensemble decision, limitations, and intended use.
Limitations
The project has several important limitations:
- The dataset is synthetic and does not represent real customers.
- OOF validation is development and model-selection evidence, not a final independent production test.
- The forward validation periods have related historical training data.
- Calibration diagnostics do not establish future calibration performance.
- The challenge dataset has no outcome labels.
- Regional diagnostics do not establish fairness.
- Day 4 explanations should not automatically be attributed to the final Day 5 model.
- Operational performance, real-world fairness, and real lending value have not been established.
Intended Use
Tamweel Lite is an educational machine learning project designed to demonstrate an end-to-end model development and evaluation workflow.
The project is not intended for real-world lending, financing, credit assessment, customer approval, rejection, or other consequential decisions.
A model decision of 1 represents only a simulated review flag within the educational exercise.
Monitoring Considerations
If a similar model were monitored in a real experimental setting, relevant checks would include:
- input and score drift,
- calibration quality,
- flag volume relative to capacity,
- predictive performance when labels become available,
- false-positive and false-negative behavior,
- regional diagnostics with appropriate denominators,
- changes in the relationship between model scores and outcomes.
Meaningful changes should be investigated before updating the model, calibration mapping, or decision policy.
Executive Summary
The final Tamweel Lite model selection retained Logistic Regression rather than deploying an ensemble. Logistic Regression achieved a mean Average Precision of 0.39166, and none of the ensemble methods passed the predefined worth-it gate.
The model, calibration mapping, and decision policy were frozen before challenge scoring. Among 2,500 unlabeled challenge requests, 330 exceeded the transported probability threshold. The full-batch 12% capacity policy retained 300 simulated review flags.
The project demonstrates the importance of leakage prevention, time-aware validation, out-of-fold evaluation, cost-sensitive decision making, calibration, capacity constraints, interpretability, and reproducibility.
This project uses synthetic data and is intended strictly for educational purposes.
