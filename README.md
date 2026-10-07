# Tamweel Lite — Advanced Machine Learning Methods

> **SDA-DSC-211 — Advanced Machine Learning Methods | أساليب تعلم الآلة المتقدمة**

## Course & Project Information

- **Course:** SDA-DSC-211 — Advanced Machine Learning Methods
- **Project:** Tamweel Lite
- **Cohort Code:** SDA-DSC-211
- **Training Context:** SDAIA Academy training programme  https://github.com/SDAIAAcademy
- **Environment:** Free Google Colab CPU
- **Data:** Synthetic course data only

> This is an independent learner repository created for the SDA-DSC-211 training project. It is **not an official SDAIA repository** and does not represent SDAIA or SDAIA Academy.

---

## نظرة عامة | Project Overview

**Tamweel Lite** is a synthetic educational machine learning project developed progressively across five labs.

The project studies a binary classification problem in which the objective is to estimate whether a synthetic loan applicant will experience a default event within 90 days of application.

Only information available at the application decision point is eligible for modeling. The project emphasizes not only predictive performance, but also leakage prevention, honest time-aware validation, out-of-fold evaluation, cost-sensitive decision making, calibration, interpretability, capacity constraints, reproducibility, and responsible use.

**المشروع تعليمي ويستخدم بيانات اصطناعية فقط.** يركز على بناء وتقييم نموذج تصنيف مع منع تسرب البيانات، والتحقق الزمني الصادق، واختيار سياسة قرار تعليمية، والمعايرة، والتفسير، وقابلية إعادة الإنتاج.

> **Responsible-use notice:** This project uses synthetic data for educational purposes only. It must not be used for real lending, financing, credit assessment, customer approval, rejection, or other consequential decisions.

---

## Problem Definition

The modeling task is to estimate whether an applicant will experience a default event within 90 days of application.

The prediction point is the moment of application. Therefore, only features that would be available at that time are permitted.

The positive class represents approximately **7.9%** of the original training data, making this an imbalanced binary classification problem.

A model output is interpreted as a probability used by a simulated review policy. A final decision value of `1` represents a **simulated review flag**, not a real financing rejection or approval.

---

## Project Objectives

The project was developed progressively across five labs:

1. Establish baseline models and compare boosting methods.
2. Detect leakage and implement honest time-aware validation.
3. Introduce imbalance handling, simulated decision cost, threshold selection, and capacity constraints.
4. Examine permutation importance, SHAP, calibration, and stability.
5. Compare single models and ensemble approaches, apply the Worth-It Gate, freeze the final procedure, and generate reproducible challenge predictions.

The final workflow preserves time ordering, target maturity, customer separation, and the published data roles where required.

---

## Architecture & Learner Journey

The project follows this workflow:

```text
Synthetic Course Data
        |
        v
Feature / Role Audit
        |
        v
Leakage Control
        |
        v
Training-only Preprocessing
        |
        v
Forward / Group-Aware Validation
        |
        v
Honest OOF Predictions
        |
        +-------------------------+
        |                         |
        v                         v
Model Comparison          Cost / Capacity Policy
        |                         |
        v                         v
Ensemble Worth-It Gate    Threshold Selection
        |                         |
        +------------+------------+
                     |
                     v
             Final Model Choice
                     |
                     v
             Reserved Calibration
                     |
                     v
             Frozen Inference
                     |
                     v
         Challenge Probabilities
                     |
                     v
        Frozen Batch Capacity Policy
                     |
                     v
             submission.csv
```

Preprocessing, imputation, weighting, and resampling are learned from training rows only. Validation, calibration, policy, evaluation, and challenge roles are kept separate according to the relevant lab design.

Challenge labels are not used or available in the learner workflow.

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
│   ├── 00_readiness_check.ipynb
│   ├── 01_baseline_boosting.ipynb
│   ├── 02_validation_tuning.ipynb
│   ├── 03_cost_sensitive_decision.ipynb
│   ├── 04_explain_calibrate.ipynb
│   ├── 05_final_model.ipynb
│   └── 99_final_check.ipynb
│
├── reports/
│   ├── DECISION_CARD.md
│   ├── INTERPRETABILITY_REPORT.md
│   ├── ENSEMBLE_DECISION.md
│   └── MODEL_CARD.md
│
├── submission/
│   ├── submission.csv
│   └── project_bundle.zip
│
├── presentation/
│   └── final_presentation.pdf
│
├── README.md
├── constraints.txt
└── requirements-colab.txt
```

The repository separates executable notebooks, generated evidence, model artifacts, reports, submission outputs, and the final presentation to support review and reproducibility.

---

## Repository Navigation

### Executed Notebooks

- [00 — Readiness Check](notebooks/00_readiness_check.ipynb)
- [01 — Baseline Boosting](notebooks/01_baseline_boosting.ipynb)
- [02 — Validation & Tuning](notebooks/02_validation_tuning.ipynb)
- [03 — Cost-Sensitive Decision](notebooks/03_cost_sensitive_decision.ipynb)
- [04 — Explain & Calibrate](notebooks/04_explain_calibrate.ipynb)
- [05 — Final Model](notebooks/05_final_model.ipynb)
- [99 — Final Check](notebooks/99_final_check.ipynb)

### Required Reports

- [Decision Card](reports/DECISION_CARD.md)
- [Interpretability Report](reports/INTERPRETABILITY_REPORT.md)
- [Ensemble Decision](reports/ENSEMBLE_DECISION.md)
- [Model Card](reports/MODEL_CARD.md)

### Final Outputs

- [Final Artifacts](artifacts/)
- [Final Model](artifacts/final_model/)
- [Final Policy](artifacts/final_policy.json)
- [Final Metrics](artifacts/final_metrics.json)
- [Submission Predictions](submission/submission.csv)
- [Final Presentation](presentation/final_presentation.pdf)

---

# Lab 1 — Baseline Boosting

## Objective

The first lab established a baseline modeling workflow and compared Logistic Regression with XGBoost and LightGBM.

The training data contained approximately **7.9% positive cases**, making the problem imbalanced.

## Model Comparison

| Model | ROC-AUC | Average Precision |
|---|---:|---:|
| Logistic Regression | 0.8213 | 0.3258 |
| XGBoost | 0.8124 | 0.3338 |
| LightGBM | 0.8138 | 0.3248 |

Logistic Regression provided competitive performance while remaining simple and fast.

Average Precision was especially useful because the positive class is relatively rare and the project is concerned with identifying positive cases rather than relying on accuracy alone.

## Initial Limitation

The initial random split was useful as a baseline but did not fully represent the temporal and repeated-customer structure of the data.

This limitation motivated the more rigorous validation strategy introduced in Lab 2.

---

# Lab 2 — Honest Validation & Tuning

## Leakage Audit

A feature-availability audit identified two post-outcome variables that should not be used as predictors:

- `days_past_due_60`
- `collection_calls`

These variables were removed because they become available after the application decision point.

A total of **22 eligible predictors** remained.

## Honest Validation

Three forward validation periods were constructed while respecting:

- chronological ordering,
- 90-day target maturity,
- customer separation,
- training-only preprocessing.

The honest validation design produced zero shared customers between training and validation within the evaluated folds.

## Leakage Effect

| Validation Scheme | Mean ROC-AUC | Mean AP |
|---|---:|---:|
| Leaky random control | 0.9999 | 0.9988 |
| Clean random control | 0.8010 | 0.3110 |
| Clean time/group fixed | 0.7976 | 0.3153 |
| Clean time/group tuned | 0.7855 | 0.3133 |

The extremely high performance of the leaky control was not realistic model quality. It demonstrated how post-outcome information can artificially inflate validation metrics.

The honest OOF evaluation covered **5,039 eligible observations**.

A bounded Optuna search was used for tuning while preserving the validation design.

---

# Lab 3 — Cost-Sensitive Decision Making

## Simulated Decision Cost

The teaching policy assigns:

- False Negative cost = **10 units**
- False Positive cost = **1 unit**

The simulated decision loss is:

`10 × FN + 1 × FP`

A maximum review capacity of **12% in each validation period** was also enforced.

The decision threshold therefore had to satisfy both predictive and operational requirements.

## Strategy Comparison

Three approaches were evaluated:

- Unweighted
- Weighted
- Oversampled

| Strategy | Pooled AP |
|---|---:|
| Unweighted | 0.3091 |
| Weighted | 0.3100 |
| Oversampled | 0.3159 |

The weighted strategy was retained according to the predefined decision procedure.

## Decision Threshold

The selected threshold was approximately:

`0.6583`

At this threshold:

- flagged requests: **526**
- recall: approximately **0.409**
- simulated decision loss: **2,639 units**
- capacity constraint: **feasible**

Capacity was checked separately within each validation period rather than only across the pooled data.

## Regional Diagnostic

Regional false-positive rates were examined descriptively.

The approximate maximum observed FPR gap was **0.65 percentage points** in the evaluated data.

These results are descriptive diagnostics only. They do not establish fairness, causality, legal compliance, or statistical significance.

---

# Lab 4 — Explainability & Calibration

## Data Roles

Separate data roles were created for:

- model fitting,
- calibration,
- policy selection,
- evaluation.

This separation reduced the risk of using the same observations for incompatible modeling decisions.

## Permutation Importance

Permutation importance identified `bureau_score` and `dti` as the strongest features in the evaluated model.

Other influential variables included:

- `loan_amount_sar`
- `savings_balance_sar`
- `prior_defaults`
- `recent_inquiries`
- `income_sar`
- `utilization_ratio`

## SHAP Analysis

SHAP was used to examine global and local model dependence.

The strongest global SHAP features included:

1. `bureau_score`
2. `dti`
3. `loan_amount_sar`
4. `savings_balance_sar`
5. `existing_obligations_sar`

The SHAP values in this analysis explain the fitted model's raw output in log-odds.

They **do not establish causality, fairness, or legal compliance**.

The Day 4 SHAP evidence applies to the Day 4 fitted model and should not automatically be attributed to the final Day 5 Logistic Regression model.

## Calibration

On the Day 4 evaluation period:

| Metric | Raw | Sigmoid |
|---|---:|---:|
| ROC-AUC | 0.7708 | 0.7708 |
| Average Precision | 0.2587 | 0.2587 |
| Brier Score | 0.1130 | 0.0671 |
| ECE | 0.1469 | 0.0225 |

Ranking performance remained unchanged while the reported probability-quality diagnostics improved in this evaluation period.

These results do not prove perfect calibration or future calibration performance.

---

# Lab 5 — Final Model & Ensemble Decision

## Nested Forward OOF Evaluation

The final model-selection procedure used nested forward out-of-fold validation.

The outer OOF evaluation contained:

**2,155 requests across three forward validation periods.**

Inner OOF predictions were used to learn ensemble components, while outer OOF predictions were used for model comparison.

The procedure preserved time ordering, label maturity, and customer separation.

## Ensemble Comparison

Three single models and three ensemble approaches were compared:

- LightGBM
- XGBoost
- Logistic Regression
- Equal averaging
- Weighted averaging
- Stacking

| Model | Mean AP | Fold SD | Mean Brier | Mean ECE |
|---|---:|---:|---:|---:|
| LightGBM | 0.34549 | 0.04348 | 0.06608 | 0.02311 |
| XGBoost | 0.35263 | 0.02904 | 0.06566 | 0.02277 |
| **Logistic Regression** | **0.39166** | **0.02981** | **0.06327** | 0.01882 |
| Equal Average | 0.37170 | 0.03258 | 0.06435 | 0.02038 |
| Weighted Average | 0.38942 | 0.02906 | 0.06332 | **0.01772** |
| Stacking | 0.38314 | 0.02949 | 0.06603 | 0.03106 |

None of the ensemble approaches produced enough improvement over the best single model to pass the predefined **Worth-It Gate**.

## Final Model Decision

**KEEP SINGLE — Logistic Regression**

Logistic Regression achieved the highest mean Average Precision:

`Mean AP = 0.39166`

The weighted ensemble came close at:

`Mean AP = 0.38942`

The additional ensemble complexity was therefore not justified by the measured OOF evidence.

`KEEP_SINGLE_MODEL` is treated as a valid engineering decision because the ensemble did not demonstrate sufficient stable added value.

---

# Final Decision Policy

The raw OOF decision threshold was approximately:

`0.16892`

The OOF policy produced an overall flag fraction of approximately:

`11.37%`

The period-level capacity checks remained feasible under the **12% review-capacity constraint**.

The teaching cost structure remained:

`FN cost = 10`  
`FP cost = 1`

The final threshold was selected from honest OOF evidence rather than from the final evaluation or challenge data.

---

# Final Calibration

The selected Logistic Regression procedure was refitted and a sigmoid calibration mapping was learned using the reserved calibration period.

The calibration set contained:

- **836 requests**
- **78 positive cases**

The final transported threshold on the calibrated probability scale was approximately:

`0.122258`

For the Lab 5 calibration fit diagnostics:

| Metric | Raw | Sigmoid |
|---|---:|---:|
| ROC-AUC | 0.789037 | 0.789037 |
| Average Precision | 0.287803 | 0.287803 |
| Brier Score | 0.076473 | 0.070858 |
| Log Loss | 0.266489 | 0.277296 |
| ECE | 0.021121 | 0.034871 |

The Brier score improved after sigmoid calibration, while ECE and log loss worsened.

Therefore, the project **does not claim that calibration improved every probability-quality metric**.

These are fit diagnostics on the reserved calibration rows used to learn the calibrator and should not be interpreted as independent evaluation or challenge performance.

---

# Challenge Predictions

The frozen model, calibration mapping, and batch decision policy were applied to all:

**2,500 unlabeled challenge requests.**

Before applying the full-batch capacity limit:

**330 requests**

exceeded the transported threshold.

The 12% full-batch capacity allowed:

**300 final review flags.**

| Item | Result |
|---|---:|
| Challenge requests | 2,500 |
| Threshold-eligible requests | 330 |
| Final capacity | 300 |
| Final review flags | 300 |
| Removed by capacity | 30 |
| Capacity satisfied | Yes |

The challenge dataset does not contain outcome labels in the learner workflow.

Therefore, challenge AP, ROC-AUC, decision loss, FPR, or other supervised performance metrics are **not reported**.

No challenge labels were used for training, tuning, calibration, model selection, or threshold selection.

---

# Final Results Summary

| Component | Final Result |
|---|---|
| Final model | Logistic Regression |
| Ensemble decision | KEEP SINGLE |
| Final selection mean AP | 0.39166 |
| Outer OOF observations | 2,155 |
| OOF raw threshold | ~0.16892 |
| Calibrated transported threshold | ~0.122258 |
| Teaching FN cost | 10 |
| Teaching FP cost | 1 |
| Review capacity | 12% |
| Challenge requests | 2,500 |
| Threshold eligible | 330 |
| Final review flags | 300 |
| Challenge labels used | No |

---

# Environment & Reproducibility

The assessed workflow is designed for the **free Google Colab CPU environment**.

It does not require:

- a paid subscription,
- a GPU,
- an external API key,
- an access token,
- or a mandatory local installation.

The recorded project environment includes:

| Component | Version / Setting |
|---|---|
| Python | 3.13.16 |
| NumPy | 2.1.3 |
| pandas | 2.2.3 |
| scikit-learn | 1.6.1 |
| XGBoost | 3.4.1 |
| LightGBM | 4.6.0 |
| SHAP | 0.52.0 |
| Optuna | 4.5.0 |
| Random seed | 211 |
| Execution | CPU |
| `n_jobs` | 2 |

Package versions, seeds, hashes, parameters, role information, and provenance evidence are preserved in the project artifacts and executed notebooks.

---

# How to Run the Project

## 1. Environment

Open the project in a clean **Google Colab CPU** environment.

Use the pinned project requirements and constraints supplied in the repository.

No paid service, API key, GPU, or Google Drive mount is required for the assessed path.

## 2. Readiness Check

Run:

`notebooks/00_readiness_check.ipynb`

Confirm that the required environment and course data are available.

## 3. Execute the Project

Run the executed workflow in order:

1. `notebooks/01_baseline_boosting.ipynb`
2. `notebooks/02_validation_tuning.ipynb`
3. `notebooks/03_cost_sensitive_decision.ipynb`
4. `notebooks/04_explain_calibrate.ipynb`
5. `notebooks/05_final_model.ipynb`

The notebooks should be reproducible using a clean **Run all** workflow in the pinned environment.

## 4. Final Verification

After all required project files are present, run:

`notebooks/99_final_check.ipynb`

The final check verifies required project files and hashes.

A green automated check supports human review. It is **not a grade, authenticity decision, or submission receipt**.

---

# Inference & Usage

The final inference workflow accepts new feature rows and produces:

- `application_id`
- probability

Probability generation occurs before applying the frozen batch policy.

The final batch policy is then applied to the generated probabilities to create the simulated review decision.

The final exported model is stored in:

`artifacts/final_model/`

The frozen decision policy is stored in:

`artifacts/final_policy.json`

Final metrics are stored in:

`artifacts/final_metrics.json`

Final challenge predictions are stored in:

`submission/submission.csv`

The exported final model is expected to reproduce the submitted probabilities from the recorded assessed repository version.

---

# Evidence & Reports

The project preserves technical evidence for the five labs under:

[`artifacts/`](artifacts/)

The four required project reports are:

1. [Decision Card](reports/DECISION_CARD.md)
2. [Interpretability Report](reports/INTERPRETABILITY_REPORT.md)
3. [Ensemble Decision](reports/ENSEMBLE_DECISION.md)
4. [Model Card](reports/MODEL_CARD.md)

These reports document the decision policy, model dependence and interpretation, ensemble decision, final model, limitations, regional diagnostics, and intended use.

---

# Data Governance & Integrity

Only the **synthetic course data** are used.

The project follows these integrity rules:

- post-outcome leakage features are excluded;
- challenge labels are not available to or used by the learner workflow;
- preprocessing is learned from training rows only;
- honest OOF predictions are used where required;
- the final threshold is not selected using challenge data;
- challenge predictions are generated by the frozen pipeline rather than manually hard-coded;
- metrics and evidence are generated from project execution rather than fabricated;
- no secrets or unnecessary personal data should be stored in the public repository.

The known post-outcome variables removed during the leakage audit were:

- `days_past_due_60`
- `collection_calls`

---

# Limitations

The project has several important limitations:

- The dataset is synthetic and does not represent real customers.
- OOF validation is development and model-selection evidence, not a final independent production test.
- The forward validation periods have related historical training data.
- Calibration diagnostics do not establish future calibration performance.
- Lab 5 calibration diagnostics are measured on the reserved calibration rows used to fit the calibrator.
- The challenge dataset has no outcome labels available to the learner.
- Challenge supervised performance therefore cannot be measured.
- Regional diagnostics do not establish fairness.
- SHAP describes model dependence and does not establish causality.
- Day 4 explanations should not automatically be attributed to the final Day 5 Logistic Regression model.
- Operational performance, legal compliance, real-world fairness, and real lending value have not been established.

---

# Responsible & Intended Use

Tamweel Lite is an educational machine learning project designed to demonstrate an end-to-end model-development and evaluation workflow.

It is **not intended for real-world lending, financing, credit assessment, customer approval, customer rejection, or other consequential decisions**.

A model decision of `1` represents only a simulated educational review flag.

The model and its interpretation outputs must not be presented as evidence of:

- causality,
- certified fairness,
- legal or regulatory compliance,
- or real-world creditworthiness.

---

# Monitoring Considerations

If a similar model were monitored in an experimental setting, relevant checks would include:

- input drift,
- score drift,
- calibration quality,
- reliability evidence,
- flag volume relative to the 12% capacity,
- predictive performance once labels become available,
- false-positive and false-negative behavior,
- regional diagnostics with appropriate denominators and positive counts,
- changes in the relationship between scores and outcomes.

Meaningful changes should be investigated before considering an update to the model, calibration mapping, or decision policy.

---

# Sources, Assistance & Disclosure

This repository was developed as part of:

**SDA-DSC-211 — Advanced Machine Learning Methods | أساليب تعلم الآلة المتقدمة**

The project follows the official Tamweel Lite course materials, lab instructions, project template, technical requirements, and administrative requirements.

External or reused material that materially affects the project should be disclosed rather than presented as original learner work.

AI assistance was used as a learning and documentation aid during the project, including support with understanding instructions, reviewing outputs, and improving documentation. Project metrics, artifacts, notebook outputs, and model evidence are based on the learner's executed course workflow and are not replaced by example or fabricated outputs.

Any approved recovery/example output, if used, must be identified as learning support and must not be presented as personal execution evidence.

## SDAIA Academy

This learner project was completed in the context of SDAIA Academy training.

**SDAIA Academy GitHub:**  
[Add the official SDAIA Academy GitHub link here]

> This is an independent learner repository. It is **not an official SDAIA or SDAIA Academy repository**, and no official endorsement or protected institutional branding is claimed.

---

# Final Presentation

The required final presentation contains **five slides** and is exported as PDF.

It must match the exact metrics and decisions in the final submitted repository.

Final presentation:

[Open Final Presentation](presentation/final_presentation.pdf)

---

# Final Version & Submission Record

The assessed version of the project is identified by the final Git tag and exact commit SHA.

- **Final Tag:** `[ADD AFTER FINAL CHECK]`
- **Exact Commit SHA:** `[ADD 40-CHARACTER SHA AFTER FINAL COMMIT]`

The final tag, exact SHA, manifest, generated bundle, notebooks, reports, model artifacts, submission output, and presentation must all correspond to the same assessed version.

The project is submitted only through the **approved private cohort submission channel**.

Submission receipts, grades, private contact information, and other unnecessary personal information are **not published in this repository**.

---

# Executive Summary | الملخص التنفيذي

## العربية

تم اختيار **KEEP SINGLE باستخدام Logistic Regression** لأن متوسط AP بلغ **0.39166** ولم ينجح أي من نماذج التجميع في تجاوز بوابة الجدوى المحددة مسبقًا.

تم تثبيت النموذج وسياسة القرار والمعايرة قبل تطبيقها على بيانات التحدي. وعند تطبيق النموذج على **2,500** طلب تحدٍ غير معلّم، تجاوز **330** طلبًا العتبة المنقولة، ثم حدّت سياسة السعة البالغة **12%** العدد النهائي إلى **300 إشارة مراجعة تعليمية**.

ركز المشروع على منع تسرب البيانات، والتحقق الزمني الصادق، وتنبؤات OOF، وتكلفة القرار التعليمية، والمعايرة، والتفسير، والسعة التشغيلية، وقابلية إعادة الإنتاج.

المشروع يستخدم بيانات اصطناعية ومخصص للتدريب فقط، ولا يصلح لاتخاذ قرارات تمويل حقيقية.

## English

The final decision was **KEEP SINGLE with Logistic Regression** because it achieved a mean Average Precision of **0.39166**, and none of the ensemble methods passed the predefined Worth-It Gate.

The model, calibration mapping, and decision policy were frozen before challenge scoring. Among **2,500 unlabeled challenge requests**, **330** exceeded the transported threshold, and the **12% full-batch capacity policy retained 300 simulated review flags**.

The project demonstrates leakage prevention, honest time-aware validation, out-of-fold evaluation, simulated cost-sensitive decision making, calibration, interpretability, operational capacity control, and reproducibility.

This project uses synthetic data and is intended strictly for educational purposes.
