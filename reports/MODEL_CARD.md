# بطاقة النموذج | Model Card
## الحالة
READY_FOR_REVIEW — جودة التفسير تحتاج مراجعة بشرية، وليست درجة آلية.
## الغرض والاستخدام | Purpose and use
This model is intended only for the Tamweel Lite synthetic educational exercise. A decision of 1 represents a simulated review flag, while a decision of 0 is not a financing approval or a safety guarantee. The model must not be used for real lending, financing, credit, or customer decisions.
بيانات Tamweel Lite اصطناعية؛ الهدف حدث خلال 90 يومًا بعد الطلب. 22 خاصية متاحة وقت الطلب؛ لا معرفات أو تواريخ أو هدف في المدخلات. الاستخدام التعليمي فقط؛ لا قرارات تمويل فعلية.
## البيانات والتحقق | Data and validation
حوض التدريب والاختيار: 6576 طلبًا. المعايرة: 836 طلبًا و78 موجبًا. حجز عملاء المعايرة، 90 يومًا لنضج التسميات، وOOF أمامي متداخل مع فصل العملاء في المستويين. طيات المقارنة: 2023Q1 و2023Q3 و2024Q1. نستبعد النتائج التي لم تنضج قبل الأدوار التالية؛ آخر بيانات التدريب لا تستخدم تلقائيًا.
Model selection used 2,155 LIVE outer-OOF predictions across three forward validation periods. The nested procedure used inner OOF predictions to learn ensemble weights and the stacker while preserving time ordering, label maturity, and customer separation. However, these three periods have overlapping training histories and the training data were already used during earlier course work, so this evidence is development and selection evidence rather than a final untouched test.
## النموذج والقرار | Model selection
KEEP SINGLE / Logistic. انحراف AP المرجعي: 0.029808. ارجع إلى artifacts/ensemble_comparison.csv وday5_fold_scores.csv للأرقام الكاملة.
The final decision was KEEP SINGLE with Logistic Regression. Logistic Regression was the best single model with mean AP 0.39166 and fold SD 0.02981. Its mean Brier score was 0.06327 and mean ECE was 0.01882. None of the Equal, Weighted, or Stack ensembles passed the predefined worth-it gate, so the additional ensemble complexity was not justified by the measured OOF evidence.
## المعايرة | Calibration
Sigmoid على عينة محجوزة من التدريب والاختيار؛ الرسم والمقاييس تشخيص على عينة تعلم المعاير، وليسا اختبارًا مستقلاً. لا ادعاء بتحسن على تحدٍّ مجهول التسميات.
The sigmoid calibration mapping was learned using the reserved July-September 2024 calibration period containing 836 rows and 78 positive cases. The reported calibration metrics are fit diagnostics measured on the same reserved rows used to learn the calibrator, so they should not be interpreted as independent evaluation or challenge performance.
## السياسة والسعة والمناطق | Policy and regions
خسارة 10 FN + FP، عتبة OOF الخام 0.16892161427109176 والمنقولة 0.12225843144286948. سعة الدفعة 300؛ المرشحون 330؛ الإشارات النهائية 300. كتلة الدرجات المتساوية لا تقسم. 1=إشارة مراجعة تعليمية، 0=عدم رفع الإشارة.
The raw OOF threshold was approximately 0.16892 and produced an OOF flag fraction of about 11.37%, which satisfied the 12% capacity constraint. On the 2,500 unlabeled challenge requests, 330 requests exceeded the transported threshold, while the full-batch 12% capacity was 300. The batch policy therefore retained 300 flags and removed 30 candidates. Regional OOF false positive rates were descriptive diagnostics only and do not establish fairness, and challenge fairness metrics cannot be calculated because challenge labels are unavailable.
## التفسير وحدوده | Explanation scope
تفسير اليوم الرابع يخص نموذج اليوم الرابع؛ لا يُنسب تلقائيًا إلى هذه النسخة. تغيير النموذج أو خصائصه أو معايرته يستلزم مراجعة التفسير.
The final model is Logistic Regression, while the Day 4 SHAP explanations were produced for the Day 4 model. Therefore, those Day 4 explanations should not automatically be attributed to this final model. Feature effects and local explanations would need to be rechecked specifically for the frozen final Logistic Regression model before making final-model explanation claims.
## المتابعة والقيود | Monitoring and limitations
If this were monitored over time, I would track input and score drift, calibration quality, flag volume against the 12% capacity limit, and performance and error rates when labels become available. I would also review regional diagnostics with their denominators and positive counts and investigate meaningful changes before considering any model, calibration, or policy update.
OOF يستخدم للاختيار، وثلاث فترات ليست اختبار دلالة. العتبة قد تتغير سعتها عند نقلها إلى نموذج معاد التدريب. لا تسميات للتحدي، ولا مقاييس أداء أو شهادة عدالة له. البيانات لا تمثل أشخاصًا أو مناطق حقيقية.
## إعادة الإنتاج | Reproducibility
seed=211; trees=80; CPU مجاني. الإصدارات في artifacts/environment.json. المصادر/بصماتها في artifacts/day5_run.json. النموذج artifacts/final_model؛ inference.predict يعيد ID واحتمالًا؛ السياسة تطبق بعد جمع الدفعة. replay_final يعيد التنبؤ المحفوظ؛ rebuild_final يعيد التدريب. لا تدرب النموذج بعد تثبيت المعاير.
## ملكيتك للتسليم | Submission ownership
أكمل أدلة الأيام السابقة والعرض، واحفظ الدفتر المنفذ، ثم سجل SHA وtag مستودعك في قناة التسليم الخاصة. دعم الدورة f486fc50dd9ac8403016facc58cf6a62beb4abf4 ليس SHA تسليمك.
