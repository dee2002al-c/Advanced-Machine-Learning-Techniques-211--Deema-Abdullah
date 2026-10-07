# بطاقة قرارك — Tamweel Lite

**الحالة:** جاهزة للمراجعة؛ لا تعني اعتمادًا أو درجة
**مصدر الأرقام:** LIVE · **الاستراتيجية:** weighted · **الصفوف:** 5,039 OOF

**المهمة:** الفئة الموجبة `default_within_90d=1` تعني حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. كل طية تحقق طلبات لاحقة، وتستبعد عملاءها من التدريب وتشترط نضج نتيجة التدريب قبل بدايتها. المعرّفات والتاريخ خارج المدخلات.

| الدليل | القيمة |
|---|---:|
| العتبة المقيدة، بالقيمة الكاملة | 0.6583471436014694 |
| عتبة أقل خسارة دون قيد | 0.44863935722081005 |
| Recall | 40.89% |
| Precision | 29.85% |
| AP مجمع منOOF | 0.3100 |
| الإشارات | 526 من 5,039 |
| FN / FP | 227 / 369 |
| الخسارة التعليمية | 2639 وحدة |
| خسارة0.5 | 2403 وحدة؛ ضمن السعة: False |
| التغير عن0.5 | +236 وحدة؛ الموجب زيادة |
| خسارة لكل10,000 طلب، تطبيع حسابي | 5237.15 وحدة |
| فجوة معدل الإنذار الخاطئ بين المناطق | 0.648 نقطة مئوية |

**القاعدة:** درجة ≥ 0.6583471436014694 تعني إشارة مراجعة داخل التمرين؛ غير ذلك بلا إشارة. لا تتخذ موافقة أو رفض تمويل حقيقي. احفظ الدقة الكاملة؛ تقريب العتبة قد يغيّر حجم الطابور.

**السياسة:** FN=10 وFP=1 وحدات تعليمية، وسعة 12% لكل فترة بعد التقريب لأسفل. ليست ريالات فعلية أو رسوم أدوات أو خصمًا من الدرجة.

**دليل السعة:** الفترة 1: 137/195, الفترة 2: 183/200, الفترة 3: 206/207.

## لماذا اخترت هذه العتبة؟
I selected a threshold of about 0.6583 for the weighted model because it was the loss-minimizing threshold that satisfied the 12% capacity limit in every validation period. It flagged 526 applications overall, achieved a recall of about 0.409, and had an observed loss of 2639 units. In comparison, the default threshold of 0.5 was not feasible under the capacity policy.

## الخسارة والسعة
The decision policy gives false negatives a cost of 10 units and false positives a cost of 1 unit, so missing a positive case is considered more costly. However, I cannot simply lower the threshold to flag more applications because only 12% of requests can be reviewed in each validation period. The selected threshold balances these two goals: reducing the defined loss while still respecting the available review capacity.

## فرق المناطق وما يحتاج إلى مراجعة
Using the same threshold for all regions, the false-positive rates were fairly close. The highest FPR was about 8.29% in the western region and the lowest was about 7.64% in the other region, giving a gap of about 0.65 percentage points. This is only a descriptive comparison on synthetic data, so it does not prove fairness or statistical significance.

## حدود النتيجة
This experiment has several limitations. The data are synthetic, and the loss values and 12% capacity limit are teaching assumptions rather than real business costs. The results are also based on three related time-based folds instead of a final independent test set. In addition, the regional comparison is descriptive only and does not include confidence intervals or a formal fairness analysis.

OOF تغطي 50.39% من التدريب و100% من الصفوف المؤهلة؛ 4,961 صفًا تمهيديًا بلا تنبؤ. اختيار العتبة وتقدير خسارتها هنا يستخدمان أهدافOOF نفسها؛ هذه نتيجة تطوير لا اختبار نهائي. لم نستخدم التحدي. المقارنة الجغرافية وصفية وليست شهادة عدالة، والأوزان لا تضمن معايرة الدرجات.

## سؤالك الأول: لماذا قد تخدعكAccuracy؟
Accuracy alone is misleading because the positive class is rare. For example, predicting no flags at all gives about 92.4% accuracy, but recall is 0, meaning that none of the positive cases are identified. This is why I also considered Average Precision, recall, decision loss, and the capacity constraint instead of relying only on accuracy.

## سؤالك الثاني: لماذا تختار علىOOF؟
I used out-of-fold predictions because each application is scored by a model that was not trained on that validation row. The same eligible OOF applications are also used when comparing the unweighted, weighted, and oversampled strategies. This gives a more honest basis for comparing the strategies and selecting the decision threshold than evaluating the models on their own training data.

أدلتك في `artifacts/threshold_metrics.json` و`day3_period_capacity.csv` و`day3_region_audit.csv` و`day3_cost_sensitivity.csv` و`cost_curve.png`. الحساسية سيناريوهات ±20% لخسارةFN، وليست فترات ثقة. راجع السعة والمعايرة عند تغير البيانات؛ لا تفترض ثباتهما مستقبلًا.
