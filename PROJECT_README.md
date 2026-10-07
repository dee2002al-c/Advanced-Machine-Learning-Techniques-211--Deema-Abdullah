# Tamweel Lite | مشروعك النهائي

## الملخص التنفيذي
تم اختيار KEEP SINGLE باستخدام Logistic Regression لأن متوسط AP بلغ 0.39166 ولم ينجح أي من نماذج التجميع في تجاوز بوابة الجدوى المحددة مسبقًا. تم تثبيت سياسة القرار والمعايرة، وعند تطبيق النموذج على 2,500 طلب تحدٍ تجاوز 330 طلبًا العتبة، ثم خفّض سقف السعة 12% العدد النهائي إلى 300 إشارة مراجعة. النموذج مخصص للتدريب على بيانات اصطناعية ولا يصلح لاتخاذ قرارات تمويل حقيقية.

## Executive summary
The final decision was KEEP SINGLE with Logistic Regression because it achieved mean AP 0.39166 and none of the ensemble methods passed the predefined worth-it gate. The decision policy and calibration were frozen before challenge scoring. Among 2,500 unlabeled challenge requests, 330 exceeded the transported threshold and the 12% full-batch capacity policy retained 300 simulated review flags. This is a synthetic educational model and is not suitable for real financing decisions.

Decision: KEEP SINGLE / Logistic. Full-batch flags: 300/2500.

اقرأ reports/MODEL_CARD.md والسياسة في artifacts/final_policy.json. الحزمة للتدريب؛ ليست نتيجة تقييم نهائية أو إثبات تسليم. ادمج أدلة أيامك السابقة واحفظ الدفتر المنفذ والعرض.
