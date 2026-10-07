# قرار التجميع

KEEP SINGLE — Logistic

The final decision was KEEP SINGLE with Logistic Regression. Logistic Regression was the best single model with mean AP 0.39166 and fold SD 0.02981. Its mean Brier score was 0.06327 and mean ECE was 0.01882. None of the Equal, Weighted, or Stack ensembles passed the predefined worth-it gate, so the additional ensemble complexity was not justified by the measured OOF evidence.

Model selection used 2,155 LIVE outer-OOF predictions across three forward validation periods. The nested procedure used inner OOF predictions to learn ensemble weights and the stacker while preserving time ordering, label maturity, and customer separation. However, these three periods have overlapping training histories and the training data were already used during earlier course work, so this evidence is development and selection evidence rather than a final untouched test.

الدليل: artifacts/ensemble_comparison.csv وday5_ensemble_gate.json. SD وصفي، وليس اختبار دلالة.
