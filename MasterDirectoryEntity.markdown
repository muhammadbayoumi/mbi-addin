إليك المرجع النهائي لجدول **`SYS_DATA_SOURCES`** (الذي يمثله الكلاس `MasterDirectoryEntity`) بعد التعديلات الأخيرة لجعله "رشيقاً" (Lean) ومعتمداً على JSON.

تم تقسيم المرجع إلى جزأين:
1.  **الأعمدة الفيزيائية:** (موجودة في قاعدة البيانات كأعمدة صريحة).
2.  **مفاتيح الـ JSON:** (موجودة داخل عمود `CONTEXT_PROPS`).

---

### أولاً: الجدول الفيزيائي (Physical Schema)
هذه الأعمدة ثابتة في الكود وقاعدة البيانات (SQLite).

| اسم العمود (Column Name) | نوع البيانات (C# / SQL) | الدور (Role) | مثال عملي (Value) |
| :--- | :--- | :--- | :--- |
| **`SOURCE_KEY`** | `String` (PK) | **المعرف الفريد للملف.**<br>يستخدمه الكود لتمييز هذا المصدر عن غيره. | `SRC_EG_STEEL_24` |
| **`TARGET_DEF_KEY`** | `String` (FK) | **الجدول المستهدف.**<br>يربط هذا الملف بجدول النظام (مثلاً: يصب في جدول الخامات). | `T_MATERIALS` |
| **`SOURCE_URI`** | `String` | **رابط الاتصال.**<br>الرابط المباشر لملف Google Sheet بصيغة TSV. | `https://docs.google...output=tsv` |
| **`VERSION_TAG`** | `String` | **إصدار البيانات.**<br>للمقارنة مع النسخة المحلية وتحديد الحاجة للتحديث. | `v2024.10.15` |
| **`DISPLAY_LABEL`** | `String` | **اسم العرض.**<br>الاسم الذي يظهر للمستخدم في القوائم المنسدلة. | `أسعار حديد عز - مصر` |
| **`DESCRIPTION`** | `String` | **الوصف.**<br>ملاحظات تظهر عند وقوف الماوس (Tooltip). | `شامل الضريبة والنقل` |
| **`MIN_LICENSE_REQ`** | `String` (Enum) | **حجب الباقة.**<br>يخفي الملف بالكامل إذا كانت باقة المستخدم أقل من هذا المستوى. | `Free`, `Premium`, `Enterprise` |
| **`IS_ACTIVE`** | `Bool` | **حالة التفعيل.**<br>لتعطيل المصدر مؤقتاً دون حذفه. | `TRUE` / `FALSE` |
| **`CONTEXT_PROPS`** | **`JObject` (JSON)** | **الحقيبة الذكية.**<br>تحتوي على كل المتغيرات (العملة، المنطقة، الأولوية، القسم). | *انظر الجدول التالي* |

---

### ثانياً: محتوى الحقيبة الذكية (JSON Dictionary)
هذه ليست أعمدة، بل مفاتيح (Keys) تكتبها داخل ملف الـ Google Sheet أو قاعدة البيانات في عمود `CONTEXT_PROPS`.
**الميزة:** يمكنك إضافة مفاتيح جديدة في المستقبل دون تعديل الكود.

| المفتاح (JSON Key) | النوع | الوصف والدور | القيم المقترحة |
| :--- | :--- | :--- | :--- |
| **`"Currency"`** | `String` | **العملة.**<br>لتحويل الأسعار تلقائياً. | `"USD"`, `"SAR"`, `"EGP"` |
| **`"Region"`** | `String` | **المنطقة/الدولة.**<br>لحساب الشحن والجمارك. | `"EG"`, `"KSA"`, `"GLOBAL"` |
| **`"Group"`** | `String` | **القسم (بديل Section_Tag).**<br>لتجميع المصادر في القائمة المنسدلة. | `"Steel"`, `"Cement"`, `"Finishes"` |
| **`"Priority"`** | `Int` | **الأولوية.**<br>1 = المصدر المفضل، 2 = المصدر البديل. | `1`, `2`, `3` |
| **`"Domain"`** | `String` | **طبيعة البيانات.**<br>توجيه معادلات الحساب (ضرب أم قسمة). | `"MATERIAL"`, `"LABOR"`, `"VENDOR"` |
| **`"Provider"`** | `String` | **اسم المورد.**<br>معلومة إضافية للتقارير. | `"Ezz Steel"`, `"RSMeans"` |
| **`"Tax_Included"`** | `Bool` | **هل يشمل الضريبة؟**<br>يمكن استخدامه في معادلات `TRANSFORM_MAPS`. | `true`, `false` |

---

### مثال كامل لصف واحد (JSON Format)

هذا ما سيتم تخزينه فعلياً في قاعدة البيانات للمصدر الواحد:

```json
{
  "SOURCE_KEY": "SRC_EG_CEMENT_01",
  "TARGET_DEF_KEY": "T_MATERIALS",
  "SOURCE_URI": "https://docs.google.com/spreadsheets/d/xxxx/export?format=tsv",
  "VERSION_TAG": "2024.1",
  "DISPLAY_LABEL": "أسمنت بورتلاندي - مصر",
  "MIN_LICENSE_REQ": "Free",
  "IS_ACTIVE": true,
  "CONTEXT_PROPS": {
    "Currency": "EGP",
    "Region": "EG",
    "Group": "Cement",
    "Priority": 1,
    "Domain": "MATERIAL",
    "Provider": "Generic Market",
    "Tax_Included": true
  }
}
```
