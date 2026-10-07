# تقريرك: التفسير والمعايرة — Tamweel Lite

**الحالة:** جاهز للمراجعة؛ لا يعني اعتمادًا أو درجة

**مصدر التفسير:** LIVE. **النموذج والمعايرة:** LIVE. **السعة:** CAPACITY_REVIEW_REQUIRED.

## النموذج والأدوار
LightGBM موزون، 80 شجرة. الهدف حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. الأدوار منفصلة زمنيًا وبالعملاء: تدريب 2516، معايرة 584 (40 موجب)، سياسة 589، تقييم 1733. الفجوات والتداخلات مستبعدة. سبق استخدام بيانات التقييم في الدورة، فهي ليست اختبارًا نهائيًا لم يمسّ.

## التفسير العام والمحلي
Permutation يقيس انخفاضAP على التقييم؛ إشارات المنطقة تُبدّل معًا. SHAP يفسر النموذج الخام بوحدةlog-odds وخلفية مسارات أشجار التدريب. base+sum(SHAP)=raw margin، ثمsigmoid للمجموع فقط. القيم ليست نقاط احتمال ولا تفسيرًا مباشرًا للنموذج المعاير.

The global explanations describe the model across many evaluation requests, while the local explanation describes one specific request. Globally, bureau_score and dti were the strongest features in both permutation importance and the SHAP summary. Locally, for request TR-009585, the main positive contributions were bureau_score, dti, and loan_amount_sar. This local result should not be treated as an explanation for every request.

The SHAP values explain the model's raw output in log-odds, not the final calibrated probability. For example, TR-009585 had a raw score of 0.9031 but a calibrated probability of 0.4795. The SHAP summary was calculated from 300 sampled evaluation requests, so the contributions should be interpreted in the context of this model and its SHAP background rather than as direct percentage changes in probability.

الطلب الاصطناعي TR-009585: الدرجة الخام 0.90308 والاحتمال المعاير 0.47952. اختير أعلى درجة داخل عينةSHAP دون استخدام النتيجة الفعلية.
- استخدم النموذج درجة ائتمانية اصطناعية عند الطلب بالقيمة 497 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+2.2693 log-odds؛ قيمة معوضة: False)
- استخدم النموذج نسبة الالتزام مع القسط المقترح إلى الدخل بالقيمة 1.2806 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+1.1981 log-odds؛ قيمة معوضة: False)
- استخدم النموذج مبلغ التمويل المطلوب بالقيمة 93437.3 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+0.2007 log-odds؛ قيمة معوضة: False)

The reason statements are based on the largest positive SHAP contributions for the selected request. For TR-009585, the three reported values were not imputed, but a positive contribution does not mean that a feature is always high or that changing it would cause the outcome to change. Correlated features can also share model signal, so feature importance and reason statements should not be interpreted as causal explanations.

## دليل المعايرة
على 1733 صفًا و139 موجب: Brier 0.113027 → 0.067112؛ ECE 0.146871 → 0.022486. عشر حاويات متساوية العرض مع أعدادها فيday4_reliability_bins.csv. AP 0.258677 → 0.258677؛ ROC-AUC 0.770804 → 0.770804. هذه نتائج هذه العينة وليست ضمانًا لتحسن مستقبلي.

Calibration was evaluated on a separate set of 1,733 requests with 139 positive cases. ROC-AUC stayed at 0.7708 and Average Precision stayed at 0.2587, while Brier score improved from 0.1130 to 0.0671, ECE improved from 0.1469 to 0.0225, and log-loss improved from 0.3580 to 0.2467. This shows better probability quality on this evaluation period, but it does not prove perfect calibration or guarantee the same result in future periods.

## الاستقرار
200 تكرارbootstrap صالح بسحب العملاء؛ فترات مئينية95% مع تثبيت النموذج والمعاير. لا تشمل تعلم النموذج أو المعايرة أو الانجراف المستقبلي، ولا تصف احتمال فرد. انحرافAP بين ربعي التقييم وصفي فقط. اختبارbureau_score±1 نُفذ؛ راجع day4_local_stability.csv.

The paired bootstrap used 200 customer-cluster replicates and kept each customer's requests together. The 95% percentile interval for the Brier change was approximately -0.0542 to -0.0374, which remained below zero in these replicates. However, the model and calibrator were kept fixed, so this interval does not include uncertainty from retraining, refitting the calibrator, or future temporal drift, and it is not an uncertainty interval for one applicant's probability.

## العتبة ومنطقة المراجعة
العتبة الخام 0.5881953696965011 اختيرت علىpolicy بخسارة10×FN+FP وسقف12% ثم نُقلت إلى 0.17331013263107387. لم تعدل باستخدام التقييم. المنطقة[0.15331, 0.19331] تشخيصية بعرض±0.02 وليست فترة ثقة. الاتحاد يحسب الطلب مرة واحدة.
- 2024Q3: السقف 100، الإشارات 97، اتحاد المراجعة 109.
- 2024Q4: السقف 107، الإشارات 109، اتحاد المراجعة 122.

The transported calibrated threshold was about 0.1733. In 2024Q3, the capacity was 100 requests, with 97 risk flags, but the union of risk flags and near-threshold cases increased the review candidate count to 109. In 2024Q4, the capacity was 107, while there were already 109 risk flags and the candidate review count increased to 122. Therefore the review zone exceeded the observed capacity in both periods, resulting in CAPACITY_REVIEW_REQUIRED. I would not change the threshold or capacity after seeing the evaluation results.

عند تجاوز السعة، وثّق الحاجة إلى تصميم سياسة جديدة على بيانات تطوير وتقييمها بدليل جديد. لا ترفع السقف ولا تقص الحالات بعد رؤية النتيجة. التفسير ليس سببية أو شهادة عدالة، والخسارة وحدات تعليمية لا رسوم أو خصم درجات. لا يستخدم هذا التمرين لتمويل حقيقي.
